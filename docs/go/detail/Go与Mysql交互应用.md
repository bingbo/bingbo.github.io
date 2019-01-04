

## 与mysql的交互应用

### 安装mysql通用库

```bash
go get github.com/go-sql-driver/mysql
```

### 引入包连接数据库操作

```go
db.gopackage main

import (
	//依赖包https://github.com/go-sql-driver/mysql
	_ "../mysql"
	"database/sql"
	"fmt"
)

func main() {
	//打开连接
	db, err := sql.Open("mysql", "root:123456@tcp(localhost:3306)/test?charset=utf8")
	checkError(err)

	//插入数据
	stmt, err := db.Prepare("insert user set name=?,password=?,email=?")
	checkError(err)

	res, err := stmt.Exec("1111", "1111", "1111")
	checkError(err)

	id, err := res.LastInsertId()
	checkError(err)

	fmt.Println(id)

	//更新数据
	stmt, err = db.Prepare("update user set name=? where id=?")
	checkError(err)

	res, err = stmt.Exec("2222", id)
	checkError(err)

	affect, err := res.RowsAffected()
	checkError(err)

	fmt.Println(affect)

	//查询数据
	rows, err := db.Query("select * from user")
	checkError(err)

	//遍历查询结果
	for rows.Next() {
		var id int
		var name string
		var password string
		var email string
		err = rows.Scan(&id, &name, &password, &email)
		checkError(err)
		fmt.Println(id, name, password, email)
	}

	//删除数据
	stmt, err = db.Prepare("delete from user where id=?")
	checkError(err)

	res, err = stmt.Exec(id)
	affect, err = res.RowsAffected()
	checkError(err)

	fmt.Println(affect)
	db.Close()

	m := map[string]string{
		"name": "aa",
		"age":  "30",
	}
	fmt.Println(m)

}

func checkError(err error) {
	if err != nil {
		panic(err)
	}
}
```


