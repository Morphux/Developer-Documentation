# Morphux Package Manager Protocol Specification

The MPM protocol is used between the Morphux Package Manager and a Morphux
Package Server. This protocol is used to get informations on packages, and
on the file database. This protocol is used above TCP.

# Encryption
Before any conversation starts, a TLS (1.2) handshake must happen. If a MPM
package is sent before that, the connection will be instantly closed.

# Header
```C
struct          s_header {
    u8_t        type;
    u16_t       size;
    u8_t        next_pkg_len;
    char[N]     next_pkg;
}               header_t;
```

| Name         | Size (Bytes) | Description                      |
|--------------|------------- |----------------------------------|
| type         | 1            | Type of the package              |
| size         | 2            | Total size of the package        |
| next_pkg_len | 1            | Size of the ```next_pkg``` field |
| next_pkg     | varies       | Hash of the next package         |

Type is an integer on 8 bits, and its possible values are:

| Value  | Name         | Description                                   |
|--------|--------------|-----------------------------------------------|
| 0x1    | AUTH         | Client request to authenticate                |
| 0x2    | AUTH_ACK     | Server acknowledgement of the authentication  |
| 0x3    | ERROR        | Error message, returned by the server         |
| 0x10   | REQ_GET_PKG  | Client request to get a package               |
| 0x11   | REQ_GET_FILE | Client request to get a file                  |
| 0x12   | REQ_GET_NEWS | Client request to get news                    |
| 0x13   | REQ_GET_CAT  | Client resquest to get categories             |
| 0x14   | REQ_GET_UPD  | Client request for update on package          |
| 0x20   | RESP_PKG     | Server's response for a package               |
| 0x21   | RESP_FILE    | Server's response for a file                  |
| 0x22   | RESP_NEWS    | Server's response for news                    |
| 0x23   | RESP_CAT     | Server's response for categories              |

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
```
struct      s_auth {
    u8_t        mpm_major_version;
    u8_t        mpm_minor_version;
};          auth_t;
```
| Name              | Size (Bytes) | Description                       |
|-------------------|------------- |-----------------------------------|
| mpm_major_version | 1            | Major version of the MPM protocol |
| mpm_minor_version | 1            | Minor version of the MPM protocol |

This package is send by a client to begin a conversation with the server.
The client must specify the version of the mpm protocol it use, for the entire
conversation.
Server respond with an AUTH_ACK package.

## 0x2: AUTH_ACK
```
struct      s_auth_ack {
    u8_t        mpm_major_version;
    u8_t        mpm_minot_version;
}           auth_ack_t;
```
| Name              | Size (Bytes) | Description                       |
|-------------------|------------- |-----------------------------------|
| mpm_major_version | 1            | Major version of the MPM protocol |
| mpm_minor_version | 1            | Minor version of the MPM protocol |

This package is send by the server to the client, in response of the a AUTH
request. If the server support the version asked by the client, it must respond
with the same values. If not, it must respond with the default protocol version.

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
| 0x3   | ERR_RES_NOT_FOUND    | A request send by the client find no result |


## 0x10: REQ_GET_PKG
```C
struct          s_req_get_pkg {
    u64_t       id;
    u8_t        state;
    u16_t       name_len;
    u16_t       categ_len;
    u16_t       version_len;
    char[N]     name;
    char[N]     category;
    char[N]     version;
}               req_get_pkg_t;
```
| Name        | Size (Bytes) | Description                       |
|-------------|--------------|-----------------------------------|
| id          | 4            | Package id                        |
| state       | 1            | State of the package (see bellow) |
| name_len    | 2            | Size of the name field            |
| categ_len   | 2            | Size of the category field        |
| version_len | 2            | Size of the version field         |
| name        | varies       | Package name                      |
| category    | varies       | Package category                  |
| version     | varies       | Version of the package            |

**State of the package**:

| Value | Name         | Description                              |
|-------|--------------|------------------------------------------|
| 0x1   | PKG_STABLE   | Package is marked as stable              |
| 0x2   | PKG_UNSTABLE | Package is marked as unstable            |
| 0x3   | PKG_DEV      | Package is marked as development version |

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

## 0x13: REQ_GET_CAT
```C
struct 		s_req_get_cat {
	u16_t		cat_len;
	u64_t[N]    categories;
}			req_get_cat_t;
```

| Name       | Size (Bytes) | Description                               |
|------------|------------- |-------------------------------------------|
| cat_len    | 2            | Size, in members, of the categories field |
| categories | varies       | Categories already known by the client    |

This package is sent by a client in order to retrieve differents categories.o

## 0x14: REQ_GET_UPD
```C
struct		s_req_get_upd {
	u64_t		pkg_len;
	u64_t[N]	packages;
}			req_get_upd_t;
```

| Name       | Size (Bytes) | Description                               |
|------------|------------- |-------------------------------------------|
| pkg_len    | 4            | Size, in members, of the packages field   |
| packages   | varies       | Array of client installed packages IDs    |

This package is sent by a client in order to get update on package if there is one.
The ```packages``` field is an array of package IDS installed by the client.

## 0x20: RESP_PKG

```C
struct      s_resp_pkg {
    u64_t       id;
    float       comp_time;
    float       inst_size;
    float       arch_size;
	u8_t		state;
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
| comp_time    | 4            | Compilation time (SBU)                             |
| inst_size    | 4            | Installation size (MB)                             |
| arch_size    | 4            | Archive size (MB)                                  |
| state        | 1            | State of the package (See GET_PKG table)           |
| category_len | 2            | Length of the category field                       |
| version_len  | 2            | Length of the version field                        |
| archive_len  | 2            | Length of the archive field                        |
| checksum_len | 2            | Length of the checksum field                       |
| dependencies_size | 2          | Size (In members) of dependencies field          |
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
    u8_t        type;
    u64_t       parent_id;
    u16_t       path_len;
    char[N]     path;
}           resp_file_t;
```

| Name      | Size (Bytes) | Description                                      |
|-----------|--------------|--------------------------------------------------|
| id        | 4            | ID of the file                                   |
| type      | 1            | Type of the file (see below)                     |
| parent_id | 4            | ID of the package that create / modify this file |
| path_len  | 2            | Length of the path field                         |
| path      | varies       | Absolute path of the file                        |

**Values for ```type``` field**:

| Value | Name             | Description        |
|-------|------------------|--------------------|
| 0x1   | FILE_TYPE_CONFIG | Configuration file |
| 0x2   | FILE_TYPE_BIN    | Binary             |
| 0x3   | FILE_TYPE_LIB    | Library            |
| 0x4   | FILE_TYPE_OTHER  | Other              |

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

## 0x23: RESP_CAT

```C
struct		s_resp_cat {
	u64_t		id;
	u64_t		parent_id;
	u16_t		name_len;
	char[N]		name;
}			resp_cat_t;
```

| Name       | Size (Bytes) | Description                 |
|------------|--------------|-----------------------------|
| id         | 4            | Id of the category          |
| parent_id  | 4            | Package ID of this category |
| name_len   | 2            | Size of the name field      |
| name       | varies       | Name of the category        |

# Examples
## REQ_GET_PKG
### Example 1:
#### English
Client request type ```REQ_GET_PKG```, asking for a package with the id ```485```

#### Code
```C
struct s_package {
	/* Header */
	u8_t	type 		= 0x10;
	/* Payload */
	u8_t	number 		= 1;
	u64_t	id 			= 485;
	u16_t	name_len 	= 0;
	u16_t	categ_len 	= 0;
	char[0]	name 		= "";
	char[0] categ 		= "";
};
```
#### Hexadecimal
```
00000000  10 01 e5 01 00 00 00 00  00 00 00 00 00 00        |..............|
0000000e
```

### Example 2:
#### English
Client request type ```REQ_GET_PKG```, asking for a package named ```vim```
in the category ```pkg```

#### Code
```C
struct s_package {
	/* Header */
	u8_t	type 		= 0x10;
	/* Payload */
	u8_t	number 		= 1;
	u64_t	id 			= 0;
	u16_t	name_len 	= 3;
	u16_t	categ_len 	= 3;
	char[3]	name 		= "vim";
	char[3] categ 		= "pkg";
};
```
#### Hexadecimal
```
00000000  10 01 00 00 00 00 00 00  00 00 03 00 03 00 76 69  |..............vi|
00000010  6d 70 6b 67                                       |mpkg|
00000014
```

### Example 3:
#### English
Client request type ```REQ_GET_PKG```, asking for two packages:

- Named 'vim' in category 'pkg'
- Named 'curses' in category 'lib'

#### Code
```C
struct s_package {
	/* Header */
	u8_t	type 		= 0x10;
	/* Payload */
	u8_t	number 		= 2;
	/* First Package */
	u64_t	id 			= 0;
	u16_t	name_len 	= 3;
	u16_t	categ_len 	= 3;
	char[3]	name 		= "vim";
	char[3] categ 		= "pkg";
	/* Second Package */
	u64_t	id 			= 0;
	u16_t	name_len 	= 6;
	u16_t	categ_len 	= 3;
	char[6]	name 		= "curses";
	char[3] categ 		= "lib";

};
```
#### Hexadecimal
```
00000000  10 02 00 00 00 00 00 00  00 00 03 00 03 00 76 69  |..............vi|
00000010  6d 70 6b 67 00 00 00 00  00 00 00 00 06 00 03 00  |mpkg............|
00000020  63 75 72 73 65 73 6c 69  62                       |curseslib|
00000029
```

## RESP_PKG
### Example 1:
#### English
Server response for a package, containing the following:

- ```id```: 234;
- ```name```: vim
- ```category```: pkg
- ```version```: 7.4
- ```comp_time```: 5.32 SBU
- ```inst_size```: 65.8MB
- ```archive```: vim-7.4.tar.gz (Size: 18.5MB) (Sum: e4ca2df7779ee7576579648eb4a48fc6a41b61cf043086ecd96aa66d6419216c)
- This package is dependent of the following package: 456, 1334

#### Code
```C
struct      s_resp_pkg {
	/* Header */
	u8_t	type 		            = 0x20;
    /* Payload */
	u8_t	    number 	            = 1;
    u64_t       id                  = 234;
    float       comp_time           = 5.32;
    float       inst_size           = 65.8;
    float       arch_size           = 18.5;
    u16_t       name_len            = 3;
    u16_t       category_len        = 3;
    u16_t       version_len         = 3;
    u16_t       archive_len         = 14;
    u16_t       checksum_len        = 64;
    u16_t       dependencies_size   = 2;
    char[3]     name                = "vim";
    char[3]     category            = "pkg";
    char[3]     version             = "7.4";
    char[14]    archive             = "vim-7.4.tar.gz";
    char[64]    checksum            = "e4ca2df7779ee7576579648eb4a48fc6a41b61cf043086ecd96aa66d6419216c";
    u64_t[2]    dependencies        = {456, 1334};

```

#### Hexadecimal
```
00000000  20 01 ea 00 00 00 00 00  00 00 71 3d aa 40 9a 99  | .........q=.@..|
00000010  83 42 00 00 94 41 03 00  03 00 03 00 0e 00 40 00  |.B...A........@.|
00000020  02 00 76 69 6d 70 6b 67  37 2e 34 76 69 6d 2d 37  |..vimpkg7.4vim-7|
00000030  2e 34 2e 74 61 72 2e 67  7a 65 34 63 61 32 64 66  |.4.tar.gze4ca2df|
00000040  37 37 37 39 65 65 37 35  37 36 35 37 39 36 34 38  |7779ee7576579648|
00000050  65 62 34 61 34 38 66 63  36 61 34 31 62 36 31 63  |eb4a48fc6a41b61c|
00000060  66 30 34 33 30 38 36 65  63 64 39 36 61 61 36 36  |f043086ecd96aa66|
00000070  64 36 34 31 39 32 31 36  63 c8 01 00 00 00 00 00  |d6419216c.......|
00000080  00 36 05 00 00 00 00 00  00                       |.6.......|
00000089
```

### Example 2:
#### English
Server response to a client information request about package "pkg/vim".
The ```libcurses``` package is a dependency of the ```vim``` package, so the
server automatically respond with it.

*Note*: The following code / binary is wrong, because just one dependency is 
following the ```vim``` package, even though the ```vim``` package count two
dependencies. Same thing for the ```curses``` package.

#### Code
```C
struct      s_resp_pkg {
	/* Header */
	u8_t	type 		            = 0x20;
    /* Payload */
	u8_t	    number 	            = 2;
    /* First Package */
    u64_t       id                  = 234;
    float       comp_time           = 5.32;
    float       inst_size           = 65.8;
    float       arch_size           = 18.5;
    u16_t       name_len            = 3;
    u16_t       category_len        = 3;
    u16_t       version_len         = 3;
    u16_t       archive_len         = 14;
    u16_t       checksum_len        = 64;
    u16_t       dependencies_size   = 2;
    char[3]     name                = "vim";
    char[3]     category            = "pkg";
    char[3]     version             = "7.4";
    char[14]    archive             = "vim-7.4.tar.gz";
    char[64]    checksum            = "e4ca2df7779ee7576579648eb4a48fc6a41b61cf043086ecd96aa66d6419216c";
    u64_t[2]    dependencies        = {456, 1334};
    /* Second Package */
    u64_t       id                  = 456;
    float       comp_time           = 3.42;
    float       inst_size           = 12;
    float       arch_size           = 25;
    u16_t       name_len            = 6;
    u16_t       category_len        = 3;
    u16_t       version_len         = 6;
    u16_t       archive_len         = 23;
    u16_t       checksum_len        = 64;
    u16_t       dependencies_size   = 3;
    char[6]     name                = "curses";
    char[3]     category            = "lib";
    char[6]     version             = "10.11B";
    char[23]    archive             = "libcurses-10.11B.tar.gz";
    char[64]    checksum            = "00d7ac638114cf2ecadee66a593def91f13293e99304cfd0f462f90cc0b330fb";
    u64_t[3]    dependencies        = {400, 234, 1056};

```


#### Hexadecimal
```
00000000  20 02 ea 00 00 00 00 00  00 00 71 3d aa 40 9a 99  | .........q=.@..|
00000010  83 42 00 00 94 41 03 00  03 00 03 00 0e 00 40 00  |.B...A........@.|
00000020  02 00 76 69 6d 70 6b 67  37 2e 34 76 69 6d 2d 37  |..vimpkg7.4vim-7|
00000030  2e 34 2e 74 61 72 2e 67  7a 65 34 63 61 32 64 66  |.4.tar.gze4ca2df|
00000040  37 37 37 39 65 65 37 35  37 36 35 37 39 36 34 38  |7779ee7576579648|
00000050  65 62 34 61 34 38 66 63  36 61 34 31 62 36 31 63  |eb4a48fc6a41b61c|
00000060  66 30 34 33 30 38 36 65  63 64 39 36 61 61 36 36  |f043086ecd96aa66|
00000070  64 36 34 31 39 32 31 36  63 c8 01 00 00 00 00 00  |d6419216c.......|
00000080  00 36 05 00 00 00 00 00  00 c8 01 00 00 00 00 00  |.6..............|
00000090  00 48 e1 5a 40 00 00 40  41 00 00 c8 41 06 00 03  |.H.Z@..@A...A...|
000000a0  00 06 00 17 00 40 00 03  00 63 75 72 73 65 73 6c  |.....@...cursesl|
000000b0  69 62 31 30 2e 31 31 42  6c 69 62 63 75 72 73 65  |ib10.11Blibcurse|
000000c0  73 2d 31 30 2e 31 31 42  2e 74 61 72 2e 67 7a 30  |s-10.11B.tar.gz0|
000000d0  30 64 37 61 63 36 33 38  31 31 34 63 66 32 65 63  |0d7ac638114cf2ec|
000000e0  61 64 65 65 36 36 61 35  39 33 64 65 66 39 31 66  |adee66a593def91f|
000000f0  31 33 32 39 33 65 39 39  33 30 34 63 66 64 30 66  |13293e99304cfd0f|
00000100  34 36 32 66 39 30 63 63  30 62 33 33 30 66 62 90  |462f90cc0b330fb.|
00000110  01 00 00 00 00 00 00 ea  00 00 00 00 00 00 00 20  |............... |
00000120  04 00 00 00 00 00 00                              |.......|
00000127
```
