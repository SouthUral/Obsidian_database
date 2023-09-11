[[Golang]]

```go
package main

  

import (

"context"

"fmt"

"log"

"os"

  

pgx "github.com/jackc/pgx/v5"

)

  

const (

queryTest = "SELECT column_name FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name = average_speed_sessions;"

queryT2 = "SELECT * FROM sh_disp.average_speed_sessions ORDER BY id limit 1;"

queryT3 = "SELECT * FROM sh_disp.average_speed_sessions WHERE id > 123000 ORDER BY id limit 20;"

)

  

func main() {

pg := Postgres{

Host: "localhost",

Port: "5432",

User: "kovalenko",

Password: "7500",

DataBaseName: "poly_bgp",

}

pg2 := Postgres{

Host: "localhost",

Port: "5471",

User: "kovalenko",

Password: "7500",

DataBaseName: "poly_bgp",

}

pg.connPg()

pg2.connPg()

res := pg.DoQuery(queryT3)

if len(res) == 0 {

fmt.Println("Нет новых записей")

return

}

lastOffset := getLastOffset(res)

pg2.DoWrite(res)

fmt.Println(lastOffset)

}

  

type Postgres struct {

Host string

Port string

User string

Password string

DataBaseName string

RecordingProcedure string

Conn *pgx.Conn

isReadyConn bool

chunk int

}

  

type LineTable struct {

ColumnName string `json:"column_name"`

DataType string `json:"data_type"`

}

  

type StructTableDB []LineTable

  

func (pg *Postgres) connPg() {

dbURL := fmt.Sprintf("postgres://%s:%s@%s:%s/%s", pg.User, pg.Password, pg.Host, pg.Port, pg.DataBaseName)

var err error

pg.Conn, err = pgx.Connect(context.Background(), dbURL)

if err != nil {

log.Printf("QueryRow failed: %v\n", err)

} else {

pg.isReadyConn = true

log.Println("Connect DB is ready:", pg.DataBaseName)

}

}

  

type rowTable map[string]any

  

func (pg *Postgres) DoWrite(data []rowTable) {

fields := GetFiled(data)

slicesData := dictionaryConverter(data, fields)

_, err := pg.Conn.CopyFrom(

context.Background(),

pgx.Identifier{"sh_disp", "average_speed_sessions"},

fields,

pgx.CopyFromRows(slicesData),

)

if err != nil {

fmt.Println(err.Error())

}

}

  

func GetFiled(data []rowTable) []string {

resSlice := []string{}

for key := range data[0] {

resSlice = append(resSlice, key)

}

return resSlice

}

  

func dictionaryConverter(data []rowTable, sliceField []string) [][]any {

res := make([][]any, 0)

for _, row := range data {

rowSlice := make([]any, 0)

for _, key := range sliceField {

  

rowSlice = append(rowSlice, row[key])

}

res = append(res, rowSlice)

}

return res

}

  

func (pg *Postgres) DoQuery(query string) []rowTable {

result := make([]rowTable, 0)

if pg.isReadyConn {

rows, err := pg.Conn.Query(context.Background(), query)

if err != nil {

fmt.Fprintf(os.Stderr, "QueryRow failed: %v\n", err)

}

for rows.Next() {

mapRes, err := pgx.RowToMap(rows)

if err != nil {

fmt.Fprintf(os.Stderr, "failed scan: %v\n", err)

}

result = append(result, mapRes)

}

}

return result

}

  

func getLastOffset(rows []rowTable) string {

lastId := len(rows) - 1

lastItem := rows[lastId]

return fmt.Sprintf("%d", lastItem["id"])

}
```