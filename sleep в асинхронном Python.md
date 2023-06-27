[[асинхронный Python]]

==asyncio.sleep()==
```Python
import asyncio

async def my_coroutine():
	print("Coroutine started")
	await asyncio.sleep(5)
	print("Coroutine finished") 


async def main(): 
	print("Main started") 
	await my_coroutine() 
	print("Main finished") 
	
asyncio.run(main())
```