# HarmonyOS RDB Database Encapsulation and Usage Practice

## About HarmonyOS 5
HarmonyOS 5 (also known as HarmonyOS Next) represents a revolutionary step in the evolution of Huawei's distributed operating system. As someone who has been following its development closely, I can attest to the remarkable improvements in this version. The system's microkernel architecture not only enhances security but also provides unprecedented flexibility in cross-device collaboration. What excites me most about HarmonyOS 5 is its focus on developer experience—the new ArkTS language, enhanced UI components, and improved debugging tools have made development much more efficient. The distributed capabilities allow us to create truly seamless experiences across different devices, from smartphones to tablets, smart TVs, and IoT devices.

## About the Author
I am a dedicated HarmonyOS developer based in China, with over three years of experience in the ecosystem. My journey with HarmonyOS began with its early versions, and I've witnessed its remarkable evolution. As a developer who has contributed to several open-source HarmonyOS projects, I've gained valuable insights into its architecture and best practices. I'm particularly passionate about data storage, database encapsulation, and code reusability, and I enjoy sharing my experiences through blogs and open-source contributions to help more developers embrace the HarmonyOS ecosystem.

## Basic Knowledge

Let's first understand the basics of SQL and RDB in HarmonyOS:

1. **What is SQL**
   - SQL (Structured Query Language) is the standard language for operating relational databases.
   - Most common operations are simple; for complex queries, search engines or ChatGPT can help.

2. **Common SQL Statements**
   1. **Create Table**
   ```sql
   CREATE TABLE user_info (
     id INTEGER PRIMARY KEY AUTOINCREMENT,  -- Primary key, auto-increment
     name TEXT,                            -- Text type
     age INTEGER,                          -- Integer type
     score REAL,                           -- Float type
     is_active INTEGER DEFAULT 0           -- Default value
   );
   ```
   2. **Insert Data**
   ```sql
   INSERT INTO user_info (name, age) VALUES ('Zhang San', 18);
   ```
   3. **Query Data**
   ```sql
   -- Query all fields
   SELECT * FROM user_info;

   -- Conditional query
   SELECT name, age FROM user_info WHERE age > 18;

   -- Sorting
   SELECT * FROM user_info ORDER BY age DESC;

   -- Limit number
   SELECT * FROM user_info LIMIT 10;
   ```
   4. **Update Data**
   ```sql
   UPDATE user_info SET age = 20 WHERE name = 'Zhang San';
   ```
   5. **Delete Data**
   ```sql
   DELETE FROM user_info WHERE age < 18;
   ```

## Preface
Recently, our project required data storage again. Using HarmonyOS RDB is a bit verbose, so I simply encapsulated a utility class to avoid writing repetitive code every time. Here are my notes, in case I need to refer back in the future.

## I. Development Background

At first, I didn't plan to encapsulate anything, but every time I used RDB, I had to write a lot of initialization, table creation, try-catch, etc. It was annoying. So I wrote an RDBUtils class for convenience.

## II. Implementation Steps

### 2.1 Creating the RDB Utility Class

```typescript
import relationalStore from '@ohos.data.relationalStore';
import common from '@ohos.app.ability.common';

// Database configuration
const STORE_CONFIG: relationalStore.StoreConfig = {
  name: 'user.db',
  securityLevel: relationalStore.SecurityLevel.S1
};

// Database version
const DB_VERSION = 2;

// SQL to create table
const SQL_CREATE_TABLE = `
  CREATE TABLE IF NOT EXISTS user_info (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    avatarUri TEXT,
    nickName TEXT,
    unionID TEXT,
    openID TEXT,
    authorizationCode TEXT,
    createTime INTEGER,
    memo TEXT,
    isSynced INTEGER DEFAULT 0
  )
`;

export class RDBUtils {
  private static instance: RDBUtils | null = null;
  private rdbStore: relationalStore.RdbStore | null = null;
  private context: common.Context;

  // Singleton pattern
  public static getInstance(context: common.Context): RDBUtils {
    if (!RDBUtils.instance) {
      RDBUtils.instance = new RDBUtils(context);
    }
    return RDBUtils.instance;
  }

  // Initialize database
  async initRDB(): Promise<void> {
    if (this.rdbStore) {
      return;
    }

    try {
      this.rdbStore = await relationalStore.getRdbStore(this.context, STORE_CONFIG);
      await this.rdbStore.executeSql(SQL_CREATE_TABLE);
      await this.checkAndUpdateVersion();
    } catch (err) {
      console.error(`Failed to initialize database: ${err}`);
      throw err;
    }
  }

  // Database upgrade
  private async checkAndUpdateVersion(): Promise<void> {
    // Check version and perform upgrade
    const currentVersion = await this.getCurrentVersion();
    if (currentVersion < DB_VERSION) {
      await this.upgradeDatabase(currentVersion);
    }
  }
}
```

### 2.2 Implementing CRUD Operations

```typescript
export class RDBUtils {
  // ... previous code ...

  // Insert data
  async insertUser(user: UserInfo): Promise<number> {
    if (!this.rdbStore) {
      throw new Error('Database not initialized');
    }

    const valueBucket: relationalStore.ValuesBucket = {
      'avatarUri': user.avatarUri || '',
      'nickName': user.nickName || '',
      'unionID': user.unionID || '',
      'openID': user.openID || '',
      'authorizationCode': user.authorizationCode || '',
      'createTime': user.createTime || Date.now(),
      'memo': user.memo || '',
      'isSynced': user.isSynced ? 1 : 0
    };

    try {
      return await this.rdbStore.insert('user_info', valueBucket);
    } catch (err) {
      console.error(`Failed to insert data: ${err}`);
      throw err;
    }
  }

  // Update data
  async updateUser(user: UserInfo): Promise<number> {
    if (!this.rdbStore || !user.id) {
      throw new Error('Invalid parameters');
    }

    const valueBucket: relationalStore.ValuesBucket = {
      'avatarUri': user.avatarUri || '',
      'nickName': user.nickName || '',
      // ... other fields
    };

    const predicates = new relationalStore.RdbPredicates('user_info');
    predicates.equalTo('id', user.id);

    try {
      return await this.rdbStore.update(valueBucket, predicates);
    } catch (err) {
      console.error(`Failed to update data: ${err}`);
      throw err;
    }
  }

  // Query data
  async queryUserById(id: number): Promise<UserInfo | null> {
    if (!this.rdbStore) {
      throw new Error('Database not initialized');
    }

    const predicates = new relationalStore.RdbPredicates('user_info');
    predicates.equalTo('id', id);

    try {
      const resultSet = await this.rdbStore.query(predicates, ['*']);
      if (resultSet.goToNextRow()) {
        const user = this.parseUserFromResultSet(resultSet);
        resultSet.close();
        return user;
      }
      resultSet.close();
      return null;
    } catch (err) {
      console.error(`Failed to query data: ${err}`);
      throw err;
    }
  }
}
```

## III. Pitfalls and Issues

- Not using singleton pattern caused memory spikes and app freezes—a painful lesson.
- Forgetting to close result sets led to memory leaks, which took a long time to debug.
- Batch inserts without transactions were extremely slow.
- Not handling null values when inserting data caused database errors.
- Not checking ID when updating data led to update failures and long debugging sessions.

## IV. Usage Example

For example, with a user table, you can insert, query, update, and delete with just a few lines of code—very convenient.

```typescript
// Initialize database
const rdbUtils = RDBUtils.getInstance(context);
await rdbUtils.initRDB();

// Insert data
const user: UserInfo = {
  avatarUri: 'https://example.com/avatar.jpg',
  nickName: 'Zhang San',
  unionID: '123456',
  openID: '789012',
  authorizationCode: 'abc123'
};
const id = await rdbUtils.insertUser(user);

// Query data
const user = await rdbUtils.queryUserById(id);
console.info(`Queried user: ${user?.nickName}`);

// Update data
if (user) {
  user.nickName = 'Li Si';
  await rdbUtils.updateUser(user);
}

// Delete data
await rdbUtils.deleteUser(id);
```

## V. Notes

- Never forget exception handling; if errors are not reported, users will be confused.
- Always close result sets when done, or you'll question your sanity.
- Don't get data types wrong; debugging will be painful.
- Use indexes wisely; use transactions for batch operations.
- Encrypt sensitive data; never let user data be exposed.

## VI. Summary

This utility works well for my own use. If you have better approaches, feel free to comment and share—let's avoid pitfalls together.

## VII. Reference Resources

- [HarmonyOS Application Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide)
- [SQLite Tutorial](https://www.runoob.com/sqlite/sqlite-tutorial.html)

## Welcome to Experience

This RDB utility class has been integrated into the HarmonyOS Developer Toolbox. Welcome to download and experience it!

[HarmonyOS Developer Toolbox](https://appgallery.huawei.com/app/detail?id=com.zyl.tool&channelId=SHARE)

---
> Author: In the World of Development
> Email: 1743914721@qq.com
> Copyright Notice: This article is an original work by the CSDN blogger, please include the original source link and this statement when reprinting. 