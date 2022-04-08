# 第2章 ChitChat 论坛

**ChitChat 的数据模型：**
- User——表示论坛的用户信息
- Session——表示论坛用户当前的登录会话
- Thread——表示论坛里面的帖子，每一个帖子都记录了多个论坛用户之间的对话
- Post——表示用户在帖子里面添加的回复

**使用 cookie 进行访问控制**
代码清单2-4 route_auth.go 文件中的 authenticate 处理器函数
```go
func authenticate(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    user, _ := data.UserByEmail(r.PostFormValue("email"))
    if user.Password == data.Encrypt(r.PostFormValue("password")){
        session := user.CreateSession()
        cookie := http.Cookie{
            Name: "_cookie",
            Value: session.Uuid,
            HttpOnly: true,
        }
        http.SetCookie(w, &cookie)
        http.Redirect(w, r, "/", 302)
    } else{
        http.Redirect(w, r, "/login", 302)
    }
}
```

代码清单2-5 util.go 文件中的 session 工具函数
```go
func session(w http.ResponseWriter, r *http.Request) (sess data.Session, err error) {
    cookie, err := r.Cookie("_cookie")
    if err == nil {
        sess = data.Session{Uuid: cookie.Value}
        if ok, _ := sess.Check(); !ok {
            err = errors.New("Invalid session")
        }
    }
    return 
}
```

数据库相关 表结构 P58

![pasted-image](images/Go_Web_ChitChat_/20220323160952.png)


