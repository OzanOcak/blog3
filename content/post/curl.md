---
title: "cURL for CRUD "
date: 2022-05-04T00:05:06-04:00
author: "ozan ocak"
tags: ["cURL", "web development", "CRUD"]
---

## BASH

```console
echo "Hello world!"
echo $?
mkdir folder && cd ./folder
vim test.sh
```

**$?** comment will return last return in the stuck which will be either 0 or 1

```shell
#!/bin/bash

set -e

function unit_test(){
	echo "running unit test"
	return 1
}
function integration_test(){
	echo "integration unit test"
	return 0
}
function e2e_test(){
	echo "e2e unit test"
	return 0
}

set +e
unit_test
set -e
integration_test
e2e_test

```

we need to give execution permission to executable file

```console
chmod +x test.sh
```

because we set -e , even though unit_test fails, the program executes

---

### returning from function

```shell
#!/bin/bash

function sound(){
	echo "how!!"
}
echo "dog says $sound"
```

---

### tar

we can not return anything apart from 0-255

```console
vim file1 file2
tar -cf files.tar file1 file2 && rm file1 file2
ls -lah
tar -tf files.tar | grep file1  // to list them
tar -xf files.tar // to extract files
tar -czvf files.tgz file1 file2   //zipped file (the least kb)
tar -xzvf files.tgz  // to extract them
```

---

## cURL

```sh
curl localhost:3000/dogs  #implicit get request
curl localhost:3000/dogs -X GET  # -o /folder/x.html
curl localhost:3000/dogs -X POST -d '{"name":"erbash", "bred":"bulldog"}' -H 'content-type: application/json'
echo $? # to check if returns 0

vim data.json # {"name":"jojo","bred":"papillion"}
curl localhost:3000/dogs -d @data.json -H 'content-type: application/json'  // -v (verbose)
curl localhost:3000/dogs -o /dev/dogs/data  # -w '%{response_code}'

curl -v localhost:8080/admin # with verbose will get 401 not authorized
curl localhost:8080/admin -u admin:secret_psswd   username:password
curl -I localhost:3000/dogs -X GET  # -I to get header, -L to redirect

curl localhost:3000/dogs -X GET  -o /folder/x.html
```

### update

```sh
curl localhost:8000/users/3 -X PUT -d '{"first_name":"tom","last_name":"cruse"}' -H 'content-type:application/json'
```

### get

```sh
curl localhost:3000/dogs -X GET  -v # you can read all the handshakes,certificates and html
```

### update

```sh
curl localhost:8000/users/3 -X PUT -d '{"first_name":"tom","last_name":"cruse"}' -H 'content-type:application/json'
```

### delete

```sh
curl localhost:8000/users/3 -X DELETE
```
