#Morphux Package Manager Protocol

# Header
```C
struct          s_header {
    u8_t        type;
}               header_t;
```
Type is an integer on 8 bits, and its possible values are:

| Value  | Name         | Description                                   |
|--------|--------------|-----------------------------------------------|
| 0x1    | AUTH         | Client request to authenticate                |
| 0x2    | AUTH_ACK     | Server acknowledgement of the authentication  |
| 0x3    | ERROR        | Error message, returned by the server         |
| 0x10   | REQ_GET_PKG  | Client request to get a package               |
| 0x11   | REQ_GET_FILE | Client request to get a file                  |
| 0x12   | REQ_GET_NEWS | Client request to get news                    |
| 0x20   | RESP_PKG     | Server's response for a package               |
| 0x21   | RESP_FILE    | Server's response for a file                  |
| 0x22   | RESP_NEWS    | Server's response for news                    |

# Payload
All the payloads share a common header:
```
struct          s_payload {
    u8_t        number;
    ...
}               payload_t;
```
The ```number``` field is used to concatenate multiple results in a same package.
It describes the number of responses present in the package.

## 0x1: AUTH

## 0x2: AUTH_ACK

## 0x3: ERROR
```C
struct      s_error {
    u8_t    error_type;
    u16_t   error_len;
    char[N] err;
}           error_t;
```

| Name       | Size (Bytes) | Description                    |
|------------|------------- |--------------------------------|
| error_type | 1            | Error type (see below)         |
| error_len  | 2            | Length of the err field        |
| err        | varies       | A string, containing the error |

**Error flags**:

| Value | Name                 | Description                                 |
|-------|----------------------|---------------------------------------------|
| 0x1   | ERR_SERVER_FAULT     | An error happened server side               |
| 0x2   | ERR_MALFORMED_PACKET | A packet send by the client is wrong        |
| 0x3   | ERR_RES_NOT_FOUND    | S request send by the client find no result |


## 0x10: REQ_GET_PKG
```C
struct          s_req_get_pkg {
    u64_t       id;
    u16_t       name_len;
    u16_t       categ_len;
    char[N]     name;
    char[N]     category;
}               req_get_pkg_t;
```
| Name      | Size (Bytes) | Description                |
|-----------|--------------|----------------------------|
| id        | 4            | Package id                 |
| name_len  | 2            | Size of the name field     |
| categ_len | 2            | Size of the category field |
| name      | varies       | Package name               |
| category  | varies       | Package category           |

This package is sent by a client, to the server. A client can either ask for a package by his id, by his name or by his name / category.

- Search by ID:
    - The ```id``` field must be set at something different than 0.
    - Both ```name_len``` and ```categ_len``` must be set at 0, since the fields they describe are not used.
- Search by name:
    - The ```id``` field must be set to 0. It it is not, the server will search by ID, and ignore the others fields.
    - The ```name_len``` field must be set to the length of the ```name``` field.
    - ```categ_len``` must be set to 0.
    - ```name``` must be a string describing the name of the package.
    - **/!\ Since packages can have the same name but different categories, it is possible that the server will respond with multiple results to this request.**
- Search by name and category:
    - The ```id``` field must be set to 0. It it is not, the server will search by ID, and ignore the others fields.
    - The ```name_len``` must be set to the length of the ```name``` field.
    - The ```categ_len``` must be set to the length of the ```category``` field.
    - ```name``` must be a string describing the name of the package.
    - ```categ``` must be a string describing the category of the package.



## 0x11: REQ_GET_FILE
```C
struct          s_req_get_file {
    u64_t       id;
    u16_t       path_len;
    char[N]     path;
}               req_get_file_t;
```
| Name     | Size (Bytes) | Description                    |
|----------|--------------|--------------------------------|
| id       | 4            | Get a file by his id           |
| path_len | 2            | Size of the path field         |
| path     | varies       | Path of the file to search for |

This package is sent by a client, to the server. A client can either ask for a file by his id or by his path.

- Search by ID:
    - The ```id``` field must be set to something different than 0.
    - The ```path_len``` field must be set to 0.
- Search by Path:
    - The ```id``` field must be set to 0. If it is not, the server will ignore the other fiels and search by ID.
    - The ```path_len``` field must be set a the length of the ```path``` field.
    - The ```path``` field must be a string, containing the path to search for.

## 0x12: REQ_GET_NEWS
```C
struct      s_req_get_news {
    time_t      last_request;
    u16_t       pkgs_ids_size;
    u64_t[N]    pkgs_ids;
}           req_get_news_t;
```
| Name          | Size (Bytes) | Description                                           |
|---------------|------------- |-------------------------------------------------------|
| last_request  | 8            | Timestamp of the last client request                  |
| pkgs_ids_size | 2            | Size of the array in pkgs_ids (In members, not bytes) |
| pkgs_ids      | varies       | Array of package ids                                  |

This package is sent by a client in order to retrieve the last development news about certain packages.

## 0x20: RESP_PKG

```C
struct      s_resp_pkg {
    u64_t       id;
    u16_t       name_len;
    u16_t       category_len;
    u16_t       version_len;
    u16_t       archive_len;
    u16_t       checksum_len;
    u16_t       dependencies_size;
    char[N]     name;
    char[N]     category;
    char[N]     version;
    char[N]     archive;
    char[N]     checksum;
    u64_t[N]    dependencies;
}           resp_pkg_t;
```

| Name         | Size (Bytes) | Description                                        |
|--------------|--------------|----------------------------------------------------|
| id           | 4            | ID of the package                                  |
| name_len     | 2            | Length of the name field                           |
| category_len | 2            | Length of the category field                       |
| version_len  | 2            | Length of the version field                        |
| archive_len  | 2            | Length of the archive field                        |
| checksum_len | 2            | Length of the checksum field                       |
| name         | varies       | Name of the package                                |
| category     | varies       | Category of the package                            |
| version      | varies       | Version of the package                             |
| archive      | varies       | Archive name of the package                        |
| checksum     | varies       | Cheksum (sha256) of the package archive            |
| dependencies | varies       | Array of packages IDs, dependencies of the package |

## 0x21: RESP_FILE

```C
struct      s_resp_file {
    u64_t       id;
    u64_t       parent_id;
    u16_t       path_len;
    char[N]     path;
}           resp_file_t;
```

| Name      | Size (Bytes) | Description                                      |
|-----------|--------------|--------------------------------------------------|
| id        | 4            | ID of the file                                   |
| parent_id | 4            | ID of the package that create / modify this file |
| path_len  | 2            | Length of the path field                         |
| path      | varies       | Absolute path of the file                        |

## 0x22: RESP_NEWS

```C
struct      s_resp_news {
    u64_t       id;
    u64_t       parent_id;
    u16_t       author_len;
    u16_t       author_mail_len;
    u16_t       text_len;
    char[N]     author;
    char[N]     author_mail;
    char[N]     text;
}           resp_news_t;
```

| Name            | Size (Bytes) | Description                             |
|-----------------|--------------|-----------------------------------------|
| id              | 4            | Id of the news                          |
| parent_id       | 4            | Id of the package concerned by the news |
| author_len      | 2            | Length of the author field              |
| author_mail_len | 2            | Length of the author_mail field         |
| text_len        | 2            | Length of the text field                |
| author          | varies       | Name of the author of the news          |
| author_mail     | varies       | Mail of the author of the news          |
| text            | varies       | Actual news content                     |
