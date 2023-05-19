---
title: go学习&selFund进展
date: 2021-08-16 11:59:32
tags:
- gin
- selFund
categories:
- go
---

这里按照时间记录一下gin框架使用的大致问题&如何使用

<!--more-->

1. 在gin里使用`get & post`  (2021/8/16)

    get示例：

        router.GET("/getUserFund/:userid", func(c *gin.Context) {
		    userid := c.Param("userid")
		    c.String(http.StatusOK, "Hello %s", userid)
	    })

    使用post取json里的数据相对麻烦一些，需要先创建对应json的结构体，然后再post方法里解析这个结构体，获取里边的数据，这个过程中也可以用gin来做一些校验之类的事情:

        type UserInfo struct {
        	User   string `form:"User" json:"User" xml:"User"  binding:"required"`
        	Fund   string `form:"Fund" json:"Fund" xml:"Fund" binding:"required"`
        	Amount string `form:"Amount" json:"Amount" xml:"Amount" binding:"-"`
        }

	    router.POST("/addUserFund", func(c *gin.Context) {
	    	var json UserInfo
	    	if err := c.ShouldBindBodyWith(&json, binding.JSON); err != nil {
	    		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
	    		return
	    	}
	    	c.String(http.StatusOK, " %s + %s", json.User, json.Fund)
	    })

    关于为什么用`ShouldBindBodyWith`是看了[这篇文章的观点](https://blog.csdn.net/yes169yes123/article/details/106204252),主要是`ShouldBindBodyWith`这个方法保存了requests的body到上下文，允许在处理方法中多次调用body；如果只调用一次的话其实可以用`ShouldBindJSON`来提高效率。

    binding里边可以指定校验方法，比如`required`指定必需，`-`指定不是必填字段等····


2. 引用其他package： (2021/08/17)

    go中引用其他package有一个前提：要在gopath中可以找到，可以用`go env | grep GOPATH`来看一下gopath是哪个文件夹。被引用的包放在这里就比较稳当··虽然感觉怪怪的，但是我的文件目录现在是这样的：

        - GOPATH
            - src
                - main.go
                - go.mod
                - service
                    - user.go


3. sql使用: (2021/8/19)

    这两天稍微看了一下如何读写数据进数据库:

    以下是基础版本：

    ```go

    //先建立表，建好之后可以直接拿建表sql生成下列结构，搜一下还是挺多的
    type Fund struct {
	    Id    int    `gorm:"column:id" db:"id" json:"id" form:"id"`
	    Name  string `gorm:"column:name" db:"name" json:"name" form:"name"`
	    Code  string `gorm:"column:code" db:"code" json:"code" form:"code"`
	    Bond  string `gorm:"column:bond" db:"bond" json:"bond" form:"bond"`
	    Cash  string `gorm:"column:cash" db:"cash" json:"cash" form:"cash"`
	    Stock string `gorm:"column:stock" db:"stock" json:"stock" form:"stock"`
	    Title string `gorm:"column:title" db:"title" json:"title" form:"title"`
	    Total string `gorm:"column:total" db:"total" json:"total" form:"total"`
    }

    //先用sql.Open()填好信息，这个时候还不会连接数据库，然后QueryRow()方法可以获取单条数据
    //用Scan()方法可以将得到的值映射到fund中，可以感觉到这里非常不美观··
    //sql.ErrNoRows 这个可以来判断是否有返回结果
    func GetFundInfo(fundCode string) Fund {
    	db, _ := sql.Open("mysql", "root:123456@tcp(localhost:3306)/fund")
    	var fund Fund
    	err := db.QueryRow("select * from fund where code = ?", fundCode).Scan(&fund.Id, &fund.Name, &fund.Code, &fund.Bond, &fund.Cash, &fund.Stock, &fund.Title, &fund.Total)
    	fmt.Println(fund)
    	defer db.Close()
    	if err == sql.ErrNoRows {
    		fundPosition := getFundPosition("https://api.doctorxiong.club/v1/fund/  position?code=" + fundCode)
    		savePosition(fundPosition, fundCode)
    	}
    	return fund
    }
    
    //写数据也类似，这里直接给一个示例;
    res, _ := db.Exec("INSERT INTO `fund`.`hold_stock` (`fund_code`, `stock_code`, `name`, `precent`, `hold`, `hold_amount`) VALUES(?,?,?,?,?,?)", fundCode, v[0], v[1], v[2], v[3], v[4])

    ```

    用了java里边的ORM中自动将entity和数据库里边table的field做了映射，看上边的代码觉得真的很繁琐···

    这里就可以用反射来帮助简化代码··至于怎么反射，我明天再看··


    两个参考资料：
        
    1. [database/sql, 和資料庫打個招呼](https://ithelp.ithome.com.tw/articles/10220392)
    2. [Go database/sql Scan & Value, 讓操作sql有一點點ORM的感覺](https://tedmax100.github.io/2020/12/21/Go-Database-Scan/)
    3. [Go Reflect 提高反射性能](https://geektutu.com/post/hpg-reflect.html)


4. 使用gorm(2021/9/2)

    今天反应了过来，为啥要自己傻乎乎的写sql？直接用框架不香吗··类似mybatsis之类，go里边也有gorm、xorm之类的ORM库，使用起来大概会方便很多吧。