# Object

```js
let user = {
  name: 'John',
  age: 20,
  isAdmin: true
}

// 访问属性
user.name
user['age']

// 删除属性
delete user.isAdmin

// 检查是否含有指定属性
'age' in user

// for...in 访问所有属性
for (const key in user){
  console.log(key)
  consolt.log(user[key])
}
```

