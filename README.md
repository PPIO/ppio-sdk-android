# PPIO-SDK-ANDROID

Android SDK for [PPIO](https://www.pp.io/docs/guide/) storage service.

## Introduction
The ppio-sdk-android is an Android Archive file. It provides an encapsulation of JSON-RPC interface.

## Getting started

### Prepare your PPIO wallet account
You must have a PPIO wallet account first to play with PPIO's products and this library. There is a [guide](https://www.pp.io/docs/wallet/) on how to generate a PPIO wallet account and get the `keystore` and `passphrase` of it.

### Start and stop service
#### Import aar
Put the aar file in the Android project's libs directory, then add the following text in the app module build.gradle:
```groovy
...
dependencies {
   implementation files('libs/poss.aar')
}
```
* poss.aar: the file name of the aar.

#### start service
You need to initialize a PPIO directroy and start the PPIO daemon service from it.
```java
try {
     JSONObject bootStrapJSONObject = new JSONObject();
     try {
         URL url = new URL("http://bootstrap.testnet.pp.io/");
         HttpURLConnection conn = (HttpURLConnection) url.openConnection();
         conn.setConnectTimeout(3 * 1000);
         conn.setRequestProperty("User-Agent", "Mozilla/4.0 (compatible; MSIE 5.0; Windows NT; DigExt)");
         InputStream inputStream = conn.getInputStream();
         String bootStrapJSONStr = readInputStream(inputStream);//change Inputstream into String
         bootStrapJSONObject = new JSONObject(bootStrapJSONStr);
     } catch (Exception e) {
         e.printStackTrace();
     }

     Config config = Poss.createDefaultConfig();

     config.setTCPPort(8068);
     config.setUDPPort(8068);
     config.setRPCPort(18068);

     config.setZone(2);

     boolean directoryExist;

     File appCacheDir = new File(APP_CACHE_DIR);

     directoryExist = appCacheDir.exists();
     if (!directoryExist) {
         directoryExist = appCacheDir.mkdir();
     }

     if (!directoryExist) {
        return false;
     }

     File cacheDir = new File(ACCOUNT_CACHE_ADDRESS_DIR);

     directoryExist = cacheDir.exists();
     if (!directoryExist) {
         directoryExist = cacheDir.mkdir();
     }

     if (directoryExist) {
         mCacheDir = cacheDir.getAbsolutePath();

         config.setDir(cacheDir.getAbsolutePath());

         config.setTestNet("test");

         String bootStrapStr = null;
         try {
              bootStrapStr = bootStrapJSONObject.getString("Bootstrap");
         } catch (JSONException e) {
              e.printStackTrace();
         }
         if (!TextUtils.isEmpty(bootStrapStr)) {
              config.setBootstrap(bootStrapStr);
         } else {
              config.setBootstrap("[\n" +
                      "  {\n" +
                      "    \"Name\": \"ali-bootstrap\",\n" +
                      "    \"IP\": \"47.110.88.167\",\n" +
                      "    \"TCPPort\": 8020,\n" +
                      "    \"UDPPort\": 8020,\n" +
                      "    \"PeerID\": \"\"\n" +
                      "  },\n" +

                      "  {\n" +
                      "    \"Name\": \"aws-bootstrap\",\n" +
                      "    \"IP\": \"54.202.181.27\",\n" +
                      "    \"TCPPort\": 8020,\n" +
                      "    \"UDPPort\": 8020,\n" +
                      "    \"PeerID\": \"\"\n" +
                      "  }\n" +
                      "]");
         }

         JSONObject payment = null;
         try {
              payment = (JSONObject) bootStrapJSONObject.get("Payment");
         } catch (Exception e) {
              e.printStackTrace();
         }

         if (payment != null) {
             config.getPayment().setIP(payment.getString("IP"));
             config.getPayment().setUDPPort(payment.getInt("UDPPort"));
             config.getPayment().setTCPPort(payment.getInt("TCPPort"));
             config.getPayment().setHTTPPort(payment.getInt("HTTPPort"));
         } else {
             config.getPayment().setIP("ad04b30b910c311e9b71c02d26ce9aff-567092461.us-west-2.elb.amazonaws.com");
             config.getPayment().setUDPPort(0);
             config.getPayment().setTCPPort(0);
             config.getPayment().setHTTPPort(18030);
         }

         JSONObject qosServerConfig = null;
         try {
              qosServerConfig = (JSONObject) bootStrapJSONObject.get("QosServerConfig");
         } catch (JSONException e) {
              e.printStackTrace();
         }

         if (qosServerConfig != null) {
             config.getQosServerConfig().setAddr(qosServerConfig.getString("Addr"));
             config.getQosServerConfig().setEnable(true);
             config.getQosServerConfig().setNetwork("udp");
             config.getQosServerConfig().setTag("ppioqos");
             config.getQosServerConfig().setDir(QOS_CACHE_DIR);
         } else {
             config.getQosServerConfig().setEnable(true);
             config.getQosServerConfig().setNetwork("udp");
             config.getQosServerConfig().setAddr("ad416ba1c124611e9a39d06111ae4d23-1840383830.us-west-2.elb.amazonaws.com:80");//if the address is incorrect, the qoslog will saved in local
             config.getQosServerConfig().setTag("ppioqos");
             config.getQosServerConfig().setDir(QOS_CACHE_DIR);
         }

         Poss.initKeyStoreData(keyStoreStr, config.getDir());

         config.setKeyPassphrase(passPhrase);
         mUser = Poss.createUser(config);
         mUser.initKeyStoreData(keyStoreStr);
         mUser.startDaemon();

         return true;
    } else {
         return false;
    }
} catch (Exception e) {
    e.printStackTrace();

    return false;
}
```
* APP_CACHE_DIR: the app's cache directory's absolute path;
* ACCOUNT_CACHE_ADDRESS_DIR: the absolute path of account's cache directory
  that exists in app's cache directory;
* QOS_CACHE_DIR: the account's qos log cache directory's absolute path;
* KEYSTORE_STR: the account's keystore's content's string;
* PASSPHRASE: the keystore's passphrase.

#### Stop service
```java
try {
     mUser.stopDaemon();
     mUser = null;
} catch (Exception e) {
     e.printStackTrace();
}
```

### Bucket

#### Create a bucket
You need to create a bucket to upload objects.
```java
mUser.createBucket(bucket);
```
* bucket: The bucket name must be no less than 3 characters long and cannot be more than 63 characters;
  it cannot contain underscores and uppercase letters; it must start with a lowercase letter or a number.

### Object
#### Put an object
```java
try {
    mUser.putObject(bucket, key, file, meta, chiPrice, copies, expires, encrypt);
} catch (Exception e) {
    e.printStackTrace();      
}
```
* bucket: Bucket name. Store objects in this bucket;
* key: Object key value. The unique index of the object within the bucket. The key length must be
  greater than 0 and less than 4096 bytes. You can use this to implement the object's directory;
* file: The file path that needs to be uploaded. Without this parameter, an empty object (which can be
  thought of as an empty folder) is created inside the bucket. Empty files (0 bytes in size) and files larger than 10G are not supported;
* meta: Ancillary data for the object. Can't exceed 4096 bytes.;
* chiPrice: Store the price of the file. The indexer will schedule the storage miners based on this    
  price, so the user's bid is reasonable, otherwise it may not be able to find a miner who is willing to provide storage services. This parameter is not required when creating an empty object;
* copies: The number of copies. Must be greater than 1 and less than or equal to 10. The default is 5.
  This parameter is not required when creating an empty object;
* expires: Expire date. The object is stored to this date and will be deleted if the user does not
  choose to extend the storage time after expiration. Supports dates in two formats, such as "2006-01-02" or "2006-01-02T15:04:05.000Z". The current time must be greater than or equal to one day, less than or equal to one year (365 days). This parameter is not required when creating an empty object;
* encrypt: Whether encrypted, must be true.

#### Get an object
```java
try {
    mUser.getObject(bucket, key, shareCode, file, chiPrice);
} catch (Exception e) {
    e.printStackTrace();
    return false;
}
```
* bucket: The bucket in which the object is located;
* key: The key value of the object;
* shareCode: Sharing code. This parameter is used when the bucket and key are empty;
* file: The file name to which the object is downloaded to the local. Absolute or relative paths can be;
* chiPrice: File download price.

#### Delete an object
```java
try {
    mUser.deleteObject(bucket, key);
} catch (Exception e) {
    e.printStackTrace();
}
```
* bucket: The bucket in which the object is located;
* key: Object key.

#### Renew an object
```java
try {
    mUser.renewObject(bucket, key, chiPrice, copies, expires);
} catch (Exception e) {
    e.printStackTrace();
}
```
* bucket: The bucket in which the object is located;
* key: Object key;
* chiPrice: New storage price. Based on this price, PPIO will re-schedule the miners who can satisfy the
  service for storage;
* copies: The number of copies. Must be greater than 1 and less than or equal to 10. The default is 5;
* expires: New expiration time. Storage time can only be extended and cannot be shortened. Do not pass   
  this parameter if you do not need to change the expiration time. (Do not attempt to keep the storage duration unchanged by passing in the old expiration time, because time errors may cause renew to fail).

#### Share an object
```java
try {
    String shareCode = mUser.shareObject(bucket, key);
} catch (Exception e) {
    e.printStackTrace();    
}
```
* bucket: The bucket in which the object is located;
* key: Object key.

#### List objects
```java
try {
    String listObjectStr = mUser.listObjects(bucket);
    JSONArray jsonArray = new JSONArray(listObjectStr);
    int length = jsonArray.length();
    for (int i = 0; i < length; i++) {
    JSONObject jsonObject = jsonArray.getJSONObject(i);
        ...
    }

    ...
    }
} catch (Exception e) {
    e.printStackTrace();            
}
```
* bucket: Bucket name.


### Task

#### List tasks
```java
try {
    String taskStr = mUser.listTasks();
    JSONArray jsonArray = new JSONArray(taskStr);
    int length = jsonArray.length();
    TaskInfo[] taskInfos = new TaskInfo[length];
    for (int i = 0; i < length; i++) {
         JSONObject jsonObject = jsonArray.getJSONObject(i);
         ...    
    }
    ...
} catch (Exception e) {
    e.printStackTrace();
}
```

#### Pause task
```java
try {
    mUser.pauseTask(taskId);
} catch (Exception e) {
    e.printStackTrace();
}
```
* taskId: Task id.

#### Resume task
```java
try {
    mUser.resumeTask(taskId);
} catch (Exception e) {
    e.printStackTrace();
}
```
* taskId: Task id.

#### Delete task
```java
try {
    mUser.deleteTask(taskId);
} catch (Exception e) {
    e.printStackTrace();
}
```
* taskId: Task id.

#### Get task progress
```java
try {
    double progress = mUser.getJobProgress(taskId);
} catch (Exception e) {
    e.printStackTrace();
}
```
* taskId: Task id.
