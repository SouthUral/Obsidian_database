[[Библиотеки Python]]
[[Шифрование данных]]

Библиотеки подходящие для шифрования данных в Python:
 - [pycryptodomex](https://www.pycryptodome.org/en/latest/src/introduction.html)
 - [cryptography](https://github.com/pyca/cryptography)
 - [pynacl](https://github.com/pyca/pynacl)

PyCrypto лучше не использовать, в ней есть уязвимости, и она не поддерживается

Пример программы с шифрованием и дешифрованием:
```python
from dataclasses import dataclass

from Cryptodome.Cipher import PKCS1_OAEP, AES
from Cryptodome.PublicKey import RSA
from Cryptodome.PublicKey.RSA import RsaKey
from Cryptodome.Random import get_random_bytes


@dataclass
class EncryptedMessage:
    nonce: bytes
    digest: bytes
    message: bytes


def generate_key_pair() -> tuple[RsaKey, RsaKey]:
    key = RSA.generate(2048)
    return key, key.publickey()


def encrypt_session_key(session_key: bytes, public_key: RsaKey) -> bytes:
    # Шифруем сессионный ключ с помощью публичного ключа
    cipher_rsa = PKCS1_OAEP.new(public_key)
    enc_session_key = cipher_rsa.encrypt(session_key)

    return enc_session_key


def decrypt_session_key(encrypted_session_key: bytes, private_key: RsaKey) -> bytes:
    # Расшифровываем сессионный ключ с помощью приватного ключа RSA
    cipher_rsa = PKCS1_OAEP.new(private_key)
    session_key = cipher_rsa.decrypt(encrypted_session_key)

    return session_key


def encrypt(data: str, session_key: bytes) -> EncryptedMessage:
    # Шифруем и подписываем сообщение с помощью алгоритма симметричного блочного шифрования (AES).
    # Для таких алгоритмов используются режимы шифрования. В этом случае используется EAX режим,
    # который позволяет одновременно шифровать блоки данных и аутентифицировать их
    cipher_aes = AES.new(session_key, AES.MODE_EAX)
    ciphertext, digest = cipher_aes.encrypt_and_digest(data.encode())

    return EncryptedMessage(
        nonce=cipher_aes.nonce,
        digest=digest,
        message=ciphertext
    )


def decrypt(encrypted: EncryptedMessage, session_key: bytes) -> str:
    # Расшифровываем сообщение с помощью сессионного ключа по алгоритму AES
    cipher_aes = AES.new(session_key, AES.MODE_EAX, encrypted.nonce)
    data = cipher_aes.decrypt_and_verify(encrypted.message, encrypted.digest)

    return data.decode()


if __name__ == '__main__':
    raw = 'onlinecinema'
    session_key = get_random_bytes(16)
    priv_key, pub_key = generate_key_pair()

    # Сторона сервера
    encrypted_session_key = encrypt_session_key(session_key, pub_key)
    encrypted_data = encrypt(raw, session_key)
    print('Зашифрованный сессионный ключ:', encrypted_session_key)
    print('Зашифрованное сообщение:', encrypted_data.message)

    # Сторона клиента
    decrypted_session_key = decrypt_session_key(encrypted_session_key, priv_key)
    print('Расшифрованное сообщение:', decrypt(encrypted_data, decrypted_session_key))
```

Расшифровка Python скрипта:
1. Получение сессионного ключа клиентом:
    - клиент успешно аутентифицировался;
    - клиент генерирует на своей стороне RSA-ключи, если их ещё нет;
    - клиент отправляет запрос на получение `session_key` и прикладывает свой публичный RSA-ключ;
    - сервер создаёт `session_key` и шифрует его с помощью публичного RSA-ключа клиента;
    - сервер отправляет шифрованный `session_key` клиенту;
    - клиент расшифровывает `session_key` и запоминает его.
2. Получение персональных данных:
    - клиент отправляет запрос на получение персональных данных серверу;
    - сервер отправляет клиенту сообщение, зашифрованное с помощью `AES` и `session_key`;
    - клиент расшифровывает сообщение с помощью полученного `session_key` и отображает информацию на экране пользователя.

Возможны два варианта схемы:

1. С одноразовым сессионным ключом. Каждый раз нужно получать сессионный ключ.
2. С сессионным ключом со временем жизни. Ключ пересоздаётся после определённого времени.

Сессионный ключ нужен, чтобы передавать зашифрованные сообщения. Для безопасной передачи сессионного ключа используется алгоритм RSA. Чтобы зашифровать сообщение с помощью RSA, нужно сгенерировать пару ключей — приватный и публичный. Публичным ключом сообщение шифруется, а приватным ключом — расшифровывается.

В примере взяли алгоритм [PKCS1_OAEP](https://pycryptodome.readthedocs.io/en/latest/src/cipher/oaep.html), который основан на RSA. Он использует [OAEP-дополнение](https://ru.wikipedia.org/wiki/%D0%9E%D0%BF%D1%82%D0%B8%D0%BC%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5_%D0%B0%D1%81%D0%B8%D0%BC%D0%BC%D0%B5%D1%82%D1%80%D0%B8%D1%87%D0%BD%D0%BE%D0%B5_%D1%88%D0%B8%D1%84%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_%D1%81_%D0%B4%D0%BE%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5%D0%BC) (padding), что повышает криптостойкость алгоритма RSA.

У такого алгоритма есть один недостаток: он может шифровать сообщения, которые по размеру не превышают несколько сотен байт. Для сессионного ключа это вполне подходит. С помощью PKCS1_OAEP и публичного RSA-ключа шифруется сессионный ключ и передаётся обратно клиенту.

Для шифрования самих данных RSA не нужен. Тут помогает симметричный алгоритм блочного шифрования [AES](https://ru.wikipedia.org/wiki/Advanced_Encryption_Standard) или Рейндал. В качестве секретного ключа используется тот самый сессионный ключ, о котором теперь знает и сервер, и клиент.

![[Auth_Sensitive_data_1_1_1629405798.png]]

Теперь осталось получить зашифрованный текст и подпись самих данных с помощью функции `encrypt_and_digest`.

Клиенту передаётся сообщение, оно содержит в себе подпись сообщения, зашифрованный текст и `nonce`. [Nonce](https://ru.wikipedia.org/wiki/Nonce) — это акроним, который переводится на русский язык как «число, которое может быть использовано один раз». Такой блок нужен в сообщении, чтобы злоумышленник не смог повторить отправку того же сообщения. Для каждого сообщения `nonce` должен быть уникальным.

Расшифровка сообщения будет такой:

1. Расшифровывается `session_key` с помощью приватного RSA-ключа.
2. Расшифровывается сообщение и проверяется подпись при указанных `session_key` и `nonce` с помощью AES+EAX.
3. Клиенту отдаётся расшифрованное сообщение.

Ещё есть алгоритм [Диффи-Хеллмана (Diffie–Hellman)](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%BE%D1%82%D0%BE%D0%BA%D0%BE%D0%BB_%D0%94%D0%B8%D1%84%D1%84%D0%B8_%E2%80%94_%D0%A5%D0%B5%D0%BB%D0%BB%D0%BC%D0%B0%D0%BD%D0%B0) или DH, который часто применяется для безопасного обмена данными между серверами. Его можно встретить в банковских сервисах для безопасной передачи номеров карт, пин-кодов и CVC/CVV-кодов. Подробнее о работе этого алгоритма можно почитать [в этой статье](https://www.securitylab.ru/analytics/478912.php?R=1).

Чаще всего для обеспечения максимальной криптостойкости смешивают разные алгоритмы. Например, в системе для передачи пин-кода может одновременно использоваться DH, RSA и AES. Такая комбинация защитит информацию о картах клиентов, пока она передаётся на ваш сервер через вереницу серверов компаний-партнёров.