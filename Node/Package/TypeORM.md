# TypeORM

## 连接

### 创建连接

```ts
// 单个连接
import { createConnection, Connection } from "typeorm";

const connection = await createConnection({
  type: "mysql",
  host: "localhost",
  port: 3306,
  username: "test",
  password: "test",
  database: "test"
});

// 多个连接
import { createConnections, Connection } from "typeorm";

const connections = await createConnections([
  {
    name: "default",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test"
  },
  {
    name: "test2-connection",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test2"
  }
]);
```

## 实体

实体是一个映射到数据库表的类, 用`@Entity()`来标记, `@Entity('sys_user')`带参数表示数据库的表名

### 主列

至少有一个主列

- `@PrimaryColumn()`, 可以指定类型, 未指定类型则从属性类型自动推断
- `@PrimaryGeneratedColumn()`, 使用自动增量值自动生成, 保存前不必分配值, `@PrimaryGeneratedColumn('uuid')`使用`uuid`自动生成

```ts
import { Entity, PrimaryGeneratedColumn } from "typeorm";

@Entity('sys_user')
export class User {
  @PrimaryGeneratedColumn()
  id: number;
  
  @PrimaryGeneratedColumn('uuid')
  uid: string;
  
  @PrimaryColumn()
  iid: number;
}
```

### 特殊列

- `@CreateDateColumn()`自动插入日期
- `@UpdateDateColumn()`保存更新会自动更新日期
- `@VersionColumn()`保存更新会自动增长版本号

### 列类型, 列选项

类型举例: int, enum

不同的数据库, 类型不一样

```ts
export enum UserRole {
    ADMIN = "admin",
    EDITOR = "editor",
    GHOST = "ghost"
}

@Entity()
export class User {

  @PrimaryGeneratedColumn()
  id: number;

  @Column({
    type: "int", 
    length: 200 
  })
  age: number;
  
  @Column({
    type: "enum",
    enum: UserRole,
    default: UserRole.GHOST
  })
  role: UserRole

}
```

常用列选项:

- type: 列类型
- name: 数据库表中的列名
- length: 长度
- select: 定义查询时是否隐藏此列, 默认`true`, 为`false`时不会显示
- default: 默认值
- unique: 唯一列
- comment: 备注

### 继承

有共同列时, 可以设置继承, 例如 id, createdAt, updatedAt等, 减少代码量

可以有单表继承, 也可以多表继承

## 关系

### 选项

- eager: true 时, 始终使用主实体加载关系
- cascade: true 时, 插入相关对象并在数据库中更新
- nullable: 是否可为空, 默认可空

### `@JoinColumn`选项

定义关系哪一侧带有外键, 可以自定义列名

```ts
@ManyToOne(type => Category)
@JoinColumn({ name: "cat_id" })
category: Category;
```

### `@JoinTable`选项

用于多对多关系, 描述一个关联表, 关联表有 typeorm 自动创建, 表示关联的实体

```ts
@ManyToMany(type => Category)
@JoinTable({
    name: "question_categories" // 此关系的联结表的表名
    joinColumn: {
        name: "question",
        referencedColumnName: "id"
    },
    inverseJoinColumn: {
        name: "category",
        referencedColumnName: "id"
    }
})
categories: Category[];
```

### 一对一

A 只有 B, B 只有 A

举例: 一个用户只有一个配置文件, 一个配置文件只属于一个用户

```ts
// Profile
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  gender: string;

  @Column()
  photo: string;
}


// User
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from "typeorm";
import { Profile } from "./Profile";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => Profile)
  @JoinColumn()
  profile: Profile;
}
```

### 一对多,多对一

A 可以有多个 B, B 只能有一个 A

举例: 一个用户可以有多张图片, 每张图片都有自己的唯一用户

```ts
// Photo
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  url: string;

  @ManyToOne(() => User, user => user.photos)
  user: User;
}

// User
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
import { Photo } from "./Photo";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Photo, photo => photo.user)
  photos: Photo[];
}
```

### 多对多

A 可以有多个 B, B 也可以有多个 A

举例: 一个问题可以设置很多种分类, 一个分类下有很多个问题

```ts
// Category
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}


// Question
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";
import { Category } from "./Category";

@Entity()
export class Question {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  text: string;

  @ManyToMany(() => Category)
  @JoinTable()
  categories: Category[];
}
```

## Repository

实体存储库, 仅限于操作具体实体

### find 选项

select

```typescript
userRepository.find({ select: ["firstName", "lastName"] });
```

relations

```ts
userRepository.find({ relations: ["profile", "photos", "videos"] });
userRepository.find({ relations: ["profile", "photos", "videos", "videos.video_attributes"] });
```

join

```ts
userRepository.find({
    join: {
        alias: "user",
        leftJoinAndSelect: {
            profile: "user.profile",
            photo: "user.photos",
            video: "user.videos"
        }
    }
});
```

where

```ts
// 适用于嵌入的实体
userRepository.find({ where: { name: { first: "Timber", last: "Saw" } } });
```

order

```ts
userRepository.find({
    order: {
        name: "ASC",
        id: "DESC"
    }
});
```

skip

```ts
userRepository.find({
    skip: 5
});
```

take

```ts
userRepository.find({
    take: 10
});
```

cache

```ts
userRepository.find({
    cache: true
});
```

lock

```ts
{ mode: "optimistic", version: number|Date }
```

运算符

```ts
const loadedPosts = await connection.getRepository(Post).find({
  title: Not("About #1"),
  likes: LessThan(10),
  likes: LessThanOrEqual(10),
  likes: MoreThan(10),
  likes: MoreThanOrEqual(10),
  title: Equal("About #2"),
  title: Like("%out #%"),
  title: ILike("%out #%"),
  likes: Between(1, 10),
  title: In(["About #2", "About #3"]),
  title: Any(["About #2", "About #3"]),
  title: IsNull(),
  likes: Raw("1 + likes = 4"),
});
```

### API

- manager
- metadata
- queryRunner
- createQueryBuilder
- hasId
- getId
- create
- merge
- preload
- save
- remove
- insert
- update
- delete
- count
- increment
- decrement
- find
- findAndCount
- findByIds
- findOne
- findOneOrFail
- query
- clear

## QueryBuilder

别名

```ts
createQueryBuilder（"user"）
```

where

```ts
createQueryBuilder("user")
  .where("user.firstName = :firstName", { firstName: "Timber" })
  .orWhere("user.lastName = :lastName", { lastName: "Saw" });
```

select

```ts
const users = await getRepository(User)
  .createQueryBuilder("user")
  .select(["user.id", "user.name"])
  .getMany();
```

having

```ts
createQueryBuilder("user")
  .having("user.firstName = :firstName", { firstName: "Timber" })
  .orHaving("user.lastName = :lastName", { lastName: "Saw" });
```

orderby

```ts
createQueryBuilder("user").orderBy({
  "user.name": "ASC",
  "user.id": "DESC"
});
```

groupby

```ts
createQueryBuilder("user")
  .groupBy("user.name")
  .addGroupBy("user.id");
```

limit

```ts
createQueryBuilder("user").limit(10);
```

offset

```ts
createQueryBuilder("user").offset(10);
```

join

```ts
const user = await createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .where("user.name = :name", { name: "Timber" })
  .getOne();

// 带条件
const user = await createQueryBuilder("user")
  .innerJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", { isRemoved: false })
  .where("user.name = :name", { name: "Timber" })
  .getOne();

// 映射不关联的实体
const user = await createQueryBuilder("user")
  .leftJoinAndMapOne("user.profilePhoto", Photo, "photo", "photo.userId=user.id")
  .where("user.name = :name", { name: "Timber" })
  .getOne();
```

raw

```ts
const photosSums = await getRepository(User)
  .createQueryBuilder("user")
  .select("user.id")
  .addSelect("SUM(user.photosCount)", "sum")
  .where("user.id = :id", { id: 1 })
  .getRawMany();

// 结果将会像这样: [{ id: 1, sum: 25 }, { id: 2, sum: 13 }, ...]
```

分页

```ts
const users = await getRepository(User)
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.photos", "photo")
  .skip(5)
  .take(10)
  .getMany();
```

加锁

```ts
const users = await getRepository(User)
  .createQueryBuilder("user")
  .setLock("pessimistic_read")
  .getMany();
```

子查询, from, where, join支持子查询

```ts
const posts = await connection
  .createQueryBuilder()
  .select("user.name", "name")
  .from(subQuery => {
    return subQuery
      .select("user.name", "name")
      .from(User, "user")
      .where("user.registered = :registered", { registered: true });
  }, "user")
  .getRawMany();
```

