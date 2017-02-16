#Morphux Package Manager Protocol

# Header
```C
struct          s_header {
    u8_t        type;
}               header_t;
```
Type is an integer on 8 bits, and its possible values are:

| Value | Name | Description |
|-------|------|-------------|
| 0x1   | AUTH | Client request to authenticate |
| 0x2   | AUTH_ACK | Server acknowledgement of the authentication  |
| 0x3   | ERROR | Error message, returned by the server |
| 0x10   | REQ_GET_PKG | Client request to get a package |
| 0x11   | REQ_GET_FILE | Client request to get a file |
| 0x12   | REQ_GET_NEWS | Client request to get news |
| 0x20   | RESP_PKG | Server's response for a package |
| 0x21   | RESP_FILE | Server's response for a file |
| 0x22   | RESP_NEWS | Server's response for news |

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
| Name | Size (Bytes) | Description |
|------|----- |-------------|
| id   | 4 |Get a package by an id. If this field is different of 0, it will be used for the request |
| name_len | 2 |Size of the name field |
| categ_len | 2 |Size of the category field |
| name | varies | Get a package by a name. This field is a name of a package. |
| category | varies | Use to get a package. Since packages can have the same name and different categories, this field is used to specify a category to search in. |

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
| Name | Size (Bytes) | Description |
|------|----- |-------------|
| id | 4 | Get a file by his id |
| path_len | 2 | Size of the path field |
| path | varies | Path of the file to search for |

This package is sent by a client, to the server. A client can either ask for a file by his id or by his path.

- Search by ID:
    - The ```id``` field must be set to something different than 0.
    - The ```path_len``` field must be set to 0.
- Search by Path:
    - The ```id``` field must be set to 0. If it is not, the server will ignore the other fiels and search by ID.
    - The ```path_len``` field must be set a the length of the ```path``` field.
    - The ```path``` field must be a string, containing the path to search for.

## 0x12: REQ_GET_NEWS

## 0x20: RESP_PKG

## 0x21: RESP_FILE

## 0x22: RESP_NEWS
