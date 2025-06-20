# HarmonyOS Cloud Functions and Cloud Database Development Practice

## 1. Overview

### 1.1 Introduction to Cloud Database
I remember when I used to work on projects, the most troublesome part was maintaining and operating the database server. Once, the server crashed in the middle of the night, and I had to get up and fix it until dawn, only to find out it was a simple configuration issue. Now, with Huawei's cloud database, these problems are solved. It not only helps us synchronize data between the client and the cloud but also supports offline access, greatly improving development efficiency. Most importantly, I no longer have to get up in the middle of the night to fix server issues, haha.

### 1.2 Main Features
1. **Flexible Synchronization Modes**
   - Cache mode: Client-side data is a subset of cloud-side data, supports automatic caching
   - Local mode: Data is stored only locally, not synchronized with the cloud

2. **Powerful Query Capabilities**
   - Supports rich predicate queries
   - Supports multi-condition chain filtering
   - Supports sorting and result set limitation
   - Supports local/cloud data queries

3. **Real-time Updates**
   - Supports data change listening
   - Supports real-time synchronization between client, cloud, and multiple devices

4. **Offline Support**
   - Supports offline data access
   - Automatically synchronizes local writes
   - Automatically syncs when the network is restored

5. **Scalability**
   - Multi-region data replication
   - Atomic batch operations
   - Consistency guarantee
   - Transaction support

6. **Security**
   - End-to-cloud full encryption
   - Permission control based on security rules
   - Triple authentication (APP, user, service)
   - Role-based permission management

### 1.3 Working Principle
1. **Data Storage Structure**
   - Based on object model
   - Data stored as objects
   - Supports multiple data types
   - Independent management of storage areas

2. **Data Operations**
   - Supports single/batch operations
   - Supports CRUD
   - Supports complex queries
   - Supports real-time synchronization

3. **Security Mechanisms**
   - Full data encryption
   - Role-based permission management
   - Access control rules
   - Data validation mechanisms

### 1.4 Typical Application Scenarios
1. **Multi-terminal Real-time Synchronization**
   - Real-time data change push
   - Data consistency across multiple devices
   - Real-time collaboration support

2. **Offline Application Support**
   - Offline data access
   - Local data modification
   - Automatic sync when network is restored

3. **Data Security Protection**
   - End-to-end encryption
   - User data isolation
   - Fine-grained permission control

### 1.5 Platform Support
1. **Client SDKs**
   - Android
   - iOS
   - Web
   - HarmonyOS (ArkTS API 9+)
   - HarmonyOS (ArkTS API 12)

2. **Server SDKs**
   - Java
   - Node.js

3. **Third-party Libraries**
   - Unity
   - Cocos

### 1.6 Introduction to Cloud Functions
Cloud functions are even more convenient—no need to worry about server management at all. I remember spending days just configuring servers for previous projects. Once, I worked 36 hours straight on server configuration, only to find out I made a mistake and had to start over. Now, with cloud functions, we only need to focus on business logic development, and Huawei Cloud handles everything else, saving time and money. Plus, it's pay-as-you-go, so you only pay for what you use—no more worrying about wasted server resources.

### 1.7 Main Features of Cloud Functions
1. **Simplified Development and Operations**
   - Provides an efficient and reliable function development and runtime framework
   - Automatically handles server configuration and management
   - Automatic code deployment
   - Automatic load balancing
   - Supports elastic scaling
   - Ensures high availability

2. **Extending Peripheral Services**
   - Acts as the core and hub of Serverless
   - Supports connecting and extending peripheral cloud services
   - Supports free combination of various services
   - Flexibly implements business logic

### 1.8 Working Principle of Cloud Functions
1. **Development Process**
   - Develop cloud functions on the AGC platform
   - Configure triggers for functions
   - Integrate SDK on the client
   - Call when trigger conditions are met

2. **Trigger Types**
   - HTTP trigger: Triggered by HTTP requests
   - Cloud database trigger: Triggered by database operations
   - Custom trigger: Configured according to business needs

### 1.9 Typical Application Scenarios for Cloud Functions
1. **User Registration Welcome**
   - Automatically triggered after user registration
   - Sends welcome messages
   - Pushes new user guides

2. **Image Processing**
   - Automatically triggered on image upload
   - Generates thumbnails
   - Image format conversion
   - Image compression processing

### 1.10 Cloud Function Platform Support
1. **Client SDKs**
   - Android
   - iOS (supports macOS)
   - Web
   - HarmonyOS (TypeScript API 6)
   - HarmonyOS (TypeScript API 9)
   - HarmonyOS (ArkTS API 9)

2. **Server SDKs**
   - Java
   - Node.js
   - REST API

3. **Third-party Libraries**
   - Unity
   - Cocos

### 1.11 Related Concepts of Cloud Functions
1. **Function**
   - Script or program running in the cloud function
   - Handles events and returns responses
   - Implements specific business logic

2. **Event Source**
   - Other services in AGC
   - Developer-defined services
   - Publishes various types of events

3. **Trigger**
   - Listens for specified events on the event source
   - Automatically calls related functions
   - Submits event data to the function for processing

4. **Alias**
   - Pointer to a specific function version
   - Supports creating multiple aliases
   - Facilitates function version management

## 2. Development Environment Preparation

### 2.1 Environment Requirements
Before starting, we need to prepare the following:
- DevEco Studio (latest version recommended; older versions may have issues. I once spent two days debugging a bug, only to find it was a version problem)
- HarmonyOS application development SDK
- Huawei developer account (make sure to prepare this in advance, or you'll be in a rush later. I once delayed a project launch because the account review wasn't approved)
- Activate Huawei Cloud services (new users get discounts, which can save a lot. I saved nearly 1000 yuan on my first project)

### 2.2 Dependency Configuration
Add these dependencies to your project's `package.json`:
```json
{
  "dependencies": {
    "@hw-agconnect/cloud-server": "^1.0.1",
    "@kit.CloudFoundationKit": "^1.0.0"
  }
}
```

## 3. Cloud Function Development Practice

### 3.1 Project Structure
Our project structure looks like this, which is quite clear:
```
tsysuserinfo/
├── function-config.json    // Cloud function configuration file
├── package.json           // Project dependency configuration
├── tsysuserinfo.ts        // Cloud function entry file
├── CloudDBZoneWrapper.ts  // Database operation wrapper
└── t_sys_userinfo.ts      // Data model definition
```

### 3.2 Configuration Files
1. **function-config.json**
```json
{
  "handler": "tsysuserinfo.myHandler",
  "functionType": 0,
  "triggers": [
    {
      "type": "http",
      "properties": {
        "enableUrlDecode": true,
        "authFlag": "true",
        "authAlgor": "HDA-SYSTEM",
        "authType": "apigw-client"
      }
    }
  ]
}
```

2. **package.json**
```json
{
  "name": "tsysuserinfo",
  "version": "1.0.0",
  "dependencies": {
    "@hw-agconnect/cloud-server": "^1.0.1"
  }
}
```

### 3.3 Data Model Definition
Let's take user information as an example, which is often used in real projects. Once, I missed a field when designing the data model, and after going live, users reported that the feature didn't work. I had to fix the code and redeploy overnight—a painful lesson:
```typescript
class t_sys_userinfo {
    user_tel: string;
    user_name: string;
    user_email: string;
    user_open_date: Date;
    user_vip: string;
    user_vip_type: string;
    user_vip_expirationdate: Date;
    user_unionid: string;
    user_openid: string;
    user_memo: string;
    user_latest_date: Date;
    avatarUri: string;

    // Get field type mapping
    getFieldTypeMap(): Map<string, string> {
        let fieldTypeMap = new Map<string, string>();
        fieldTypeMap.set('user_tel', 'String');
        fieldTypeMap.set('user_name', 'String');
        // ... other field mappings
        return fieldTypeMap;
    }

    // Get primary key list
    getPrimaryKeyList(): string[] {
        return ['user_unionid'];
    }
}
```

### 3.4 Database Operation Wrapper
To make things easier, we encapsulated all database operations, making it very convenient to use. Once, a project went live without proper encapsulation, resulting in duplicated database operations everywhere. Changing one place meant changing many places—a real headache:
```typescript
import { cloud, CloudDBCollection } from '@hw-agconnect/cloud-server';

export class CloudDbZoneWrapper {
    collection: CloudDBCollection<t_sys_userinfo>;
    private static readonly ZONE_NAME = "db";

    constructor() {
        this.collection = cloud.database({ zoneName: CloudDbZoneWrapper.ZONE_NAME })
            .collection(t_sys_userinfo);
    }

    // Query user info
    async queryuserinfo(user_unionid: string) {
        let query = this.collection.query()
            .equalTo('user_unionid', user_unionid);
        return await query.get();
    }

    // Update or insert user info
    async upsertuserinfo(records: t_sys_userinfo, logger) {
        try {
            await this.collection.upsert(records);
            return [records];
        } catch (e) {
            logger.error('Operation failed:', e);
            throw e;
        }
    }

    // Delete user info
    async deleteuserinfo(records: t_sys_userinfo[]) {
        return await this.collection.delete(records);
    }
}
```

### 3.5 Cloud Function Implementation
The main processing logic of the cloud function, where we handle basic CRUD operations. Once, I forgot to handle exceptions, resulting in user data loss and many complaints—so error handling is crucial:
```typescript
import { CloudDbZoneWrapper } from './CloudDBZoneWrapper';

let myHandler = async function (event, context, callback, logger) {
    logger.info(event);
    let data;
    let body = JSON.parse(event.body);
    let operation = body?.operation;
    let records = body?.records;
    let cloudDBZoneWrapper = new CloudDbZoneWrapper();

    try {
        switch (operation) {
            case "query":
                data = await cloudDBZoneWrapper.queryuserinfo(records?.user_unionid);
                break;
            case "upsert":
                data = await cloudDBZoneWrapper.upsertuserinfo(records, logger);
                break;
            case "delete":
                data = await cloudDBZoneWrapper.deleteuserinfo(records);
                break;
            default:
                throw new Error("Unsupported operation type");
        }

        return callback({
            ret: { code: 0, desc: "Operation successful" },
            data
        });
    } catch (error) {
        return callback({
            ret: { code: -1, desc: error.message }
        });
    }
};

export { myHandler };
```

## 4. Client Call Example

### 4.1 Login Page Call Example
After a successful login, we need to sync user info to the cloud, which is common in real projects. Once, I forgot to handle login failure, resulting in users being unable to log in and many complaints:
```typescript
import { cloudFunction } from '@kit.CloudFoundationKit';

async function updateUserInfo(userInfo: UserInfo) {
    try {
        // Prepare user data
        let cloudUserInfo = new t_sys_userinfo();
        cloudUserInfo.user_name = userInfo.nickName;
        cloudUserInfo.user_unionid = userInfo.unionID;
        cloudUserInfo.user_openid = userInfo.openID;
        cloudUserInfo.avatarUri = userInfo.avatarUri;

        // Call cloud function
        let result = await cloudFunction.call({
            name: 'tsysuserinfo',
            data: {
                operation: 'upsert',
                records: cloudUserInfo
            }
        });

        if (result.result?.ret?.code === 0) {
            console.info('User info synced successfully');
        } else {
            throw new Error(`Sync failed: ${result.result?.ret?.desc}`);
        }
    } catch (error) {
        console.error('Failed to sync user info:', error);
        throw error;
    }
}
```

### 4.2 Call Flow Description
The entire call flow is as follows (here's a simple diagram). Once, I drew the flowchart wrong, and a new colleague followed it, resulting in buggy code and criticism from the boss:
1. After successful login, obtain the user's basic info
2. Package this info into a cloud database object
3. Call the cloud function to sync data
4. Determine if the sync was successful based on the return result

## 5. Development Suggestions

### 5.1 Data Model Design
When designing data models, I suggest:
- Use TypeScript classes for definition, which provides code hints and reduces errors. I once spent two days debugging due to poor type definitions
- Remember to implement field type mapping—very important. I once forgot, resulting in messy data
- Set primary keys and indexes properly, or queries will be slow. Our project once had snail-like query speeds due to this
- Do data validation to avoid dirty data. Once, a user entered special characters and crashed the database

### 5.2 Error Handling
When handling errors, note:
- Use try-catch to catch possible errors—very important. I had frequent crashes due to poor error handling
- Log errors for troubleshooting. Once, an online issue took ages to debug due to missing logs
- Provide user-friendly error messages. Our product manager often complained about confusing error prompts
- Consider adding retry mechanisms for poor network conditions. I had many sync failures due to lack of retries

### 5.3 Performance Optimization
Some suggestions for improving performance:
- Use data caching wisely to save traffic. After adding caching, our project's traffic costs halved
- Batch data operations for higher efficiency. I once improved performance tenfold by optimizing batch processing
- Optimize query conditions—don't fetch unnecessary data. I once crashed the server by querying too much data
- Control sync frequency—don't sync too often. Once, frequent syncing drained users' phone batteries

### 5.4 Security
Pay attention to security:
- Implement access control to prevent unauthorized access. Our project was once maliciously accessed
- Encrypt sensitive data, such as passwords. I was warned by the security team for not encrypting
- Use secure communication protocols—don't use http. Once, data was intercepted due to http
- Validate data to prevent injection attacks. Our project was once hit by SQL injection

## 6. Common Issues

### 6.1 Cloud Function Call Failure
If you encounter call failures, check:
- Network connection—sometimes it's just a network issue. I once spent a whole day debugging due to network problems
- Cloud function configuration, especially permissions. Once, I set permissions wrong and couldn't call the function
- Parameter format—this often causes errors. I once spent ages debugging due to incorrect parameter format
- Permission settings—don't set permissions too wide. Once, I almost had a security issue due to overly broad permissions

### 6.2 Data Sync Issues
If data sync fails, check:
- Data model definition—field types must match. I once failed to sync data due to type mismatch
- Primary key settings—don't duplicate. Once, duplicate keys messed up all the data
- Operation permissions—sometimes it's just a permission issue. I once couldn't update data due to insufficient permissions
- Data format—especially date formats. Once, a wrong date format caused sync failure

### 6.3 Performance Issues
If you encounter performance issues:
- Optimize query conditions—don't fetch too much data. I once crashed the server by querying too much
- Implement data pagination—don't fetch too much at once. Once, lack of pagination caused query timeouts
- Add appropriate indexes—queries will be much faster. After adding indexes, query speed increased a hundredfold
- Control data volume—don't store too much useless data. Once, storing too many images filled up storage

## 7. Summary

This document introduces how to use Huawei's cloud functions and cloud database services in HarmonyOS applications. These services have indeed solved many problems for us, especially server operations and maintenance, saving a lot of effort. In the past, a dedicated person was needed just for server maintenance; now, with cloud services, one person can handle several projects at once.

In actual development, I suggest:
1. Design data models reasonably—this is very important, and I've learned the hard way
2. Handle errors properly—don't let the program crash, or users will complain
3. Pay attention to data security—don't leak user information, or you'll get complaints
4. Focus on performance optimization—users will have a better experience and give more positive feedback

## 8. Reference Resources

- [Huawei Cloud Function Development Documentation](https://developer.huawei.com/consumer/cn/doc/AppGallery-connect-Guides/agc-cloud-function-introduction-0000001059279544) 