# Securely share your files with selfmanage-sftp

## Problem statement

Explore many other services which provide the specification to share the file without changing the protocol but problem I come across are
  - Maintenance, configuration management, security, compliances etc.
  - Not Flexible deployment model.
  - Not durability on computing or data storage.
  - License is very costly [ manage services like Aws SFTP ]

> To solve this issue we need something that runs as self-managed, less cost and provides data persistency in case of any disaster and with all standard security. I evaluated many other solutions over the internet but didn't found anything useful that can solve the above-listed issues until I found a solution by [Atmoz] at [Github] (https://github.com/atmoz/sftp) off course it's has its own area of limitation. So need a custom solution where the data and compute can be segregate in two pieces so if the compute lost data will be present in persistence data storage service like s3 or efs or nfs etc.

## Manage sftp service.

### Features

  - Complute and data storage are separate. 
  - Self-managed into a container. 
  - Support NFS, EFS or durable Amazon S3 for persistent data storage. 
  - Different users can write to different buckets or data volumes as specified. 
  - Cost-effective and you pay only for the use of the service. 
  - SFTP User/Password and Public Key Authentication. 
  - Flexible deployment model 
  - Configurable TCP ports 
  - Support commonly used clients include WinSCP, FileZilla, CyberDuck, lftp, and OpenSSH clients. 

### Usage

- Define users as command arguments, STDIN or mounted or env `USERS`
  (syntax: `USERS=user:pass[:e][:uid[:gid[:dir1][:s3_bucket_name`).
  - Set UID/GID manually for your users if you want them to make changes to
    your mounted volumes with permissions matching your host filesystem.
  - Add directory names at the end, if you want to create them and/or set user
    ownership. Perfect when you just want a fast way to upload something without
    mounting any directories, or you want to make sure a directory is owned by
    a user (chown -R).


### Simplest docker run example

```

docker run -p 2222:22 -e USERS="innus:password123:::your_directory" --rm --cap-add SYS_ADMIN --device /dev/fuse selfmanage-sftp:v2

```
User "innus" with password "password123" can login with sftp and upload files to a folder in that container at /home/innus/your_directory.
In case of s3: Files uploaded this way are synced to S3 in the named S3_BUCKET_NAME/your_directory

### Example Login: connect to a locally-running instance on the default docker IP:
```
sftp -P 21 innus@localhost:your_directory
sftp> cd your_directory
sftp> put somefile
sftp> ls
somefile
```

### Store users in Environment variable - NOTE: Multiple users is supported 
```
USERS=user1:password:::your_directory:s3_bucket_name
USERS=user1:encrypted_password:e::your_directory:s3_bucket_name
USERS=user1:password:::your_directory:s3_bucket_name

USERS=user1:password:::your_directory:s3_bucket_name;user3:password:::your_directory:s3_bucket_name;user3:password:::your_directory:s3_bucket_name

```

## Encrypted password

Add `:e` behind password to mark it as encrypted. Use single quotes if using terminal.

```
 docker run \
    -p 2222:22 \
    -v /host/share:/home/user1/your_directory \
    -e USERS="user1:encrypted_password:::your_directory" \
    --rm --cap-add SYS_ADMIN \
    --device /dev/fuse -d selfmanage-sftp:v1
```

Tip: you can use [atmoz/makepasswd](https://hub.docker.com/r/atmoz/makepasswd/) to generate encrypted passwords:  
`echo -n "your-password" | docker run -i --rm atmoz/makepasswd --crypt-md5 --clearfrom=-`

## Using SSH key (and no password)

Add environment variable SSH_KEYS - Mount all public keys in the user's will automatically be appended to `.ssh/authorized_keys`.

```
SSH_KEYS="user1@ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCNXk5K9oRMnk70qtcO7/DaoXBMAOz29pWwIGi97Ix8Akkg5XGEKmrdYpCnknkwKeCXUqWdvRnUz3GCNc02IgSSBXrVgeGl8eM/uhcjwE45sGeBiIIi6YX4qMsVH/11puD6JOcrSD6YVbRjx+q8/ArBHtZagYngKChJNDE4QN4sjQ==;user2@ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCNXk5K9oRMnk70qtcO7/DaoXBMAOz29pWwIGi97Ix8Akkg5XGEKmrdYpCnknkwKeCXUqWdvRnUz3GCNc02IgSSBXrVgeGl8eM/uhcjwE45sGeBiIIi6YX4qMsVH/11puD6JOcrSD6YVbRjx+q8/ArBHtZagYngKChJNDE4QN4sjQ=="
```

```
 docker run \
    -p 2222:22 \
    -e SSH_KEYS="innus@ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCNXk5K9oRMnk70qtcO7/DaoXBMAOz29pWwIGi97Ix8Akkg5XGEKmrdYpCnknkwKeCXUqWdvRnUz3GCNc02IgSSBXrVgeGl8eM/uhcjwE45sGeBiIIi6YX4qMsVH/11puD6JOcrSD6YVbRjx+q8/ArBHtZagYngKChJNDE4QN4sjQ==;chinna@ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCNXk5K9oRMnk70qtcO7/DaoXBMAOz29pWwIGi97Ix8Akkg5XGEKmrdYpCnknkwKeCXUqWdvRnUz3GCNc02IgSSBXrVgeGl8eM/uhcjwE45sGeBiIIi6YX4qMsVH/11puD6JOcrSD6YVbRjx+q8/ArBHtZagYngKChJNDE4QN4sjQ==" \
    -e USERS="user1:password_1:::your_directory_1;user2:password_2:::your_directory_2" \
    --rm --cap-add SYS_ADMIN \
    --device /dev/fuse -d selfmanage-sftp:v1
```

## Run with Amazon durable S3 storage

To run need to be passed an additional environment variable S3_BUCKET=true


```
 docker run \
    -p 2222:22 \
    -e SSH_KEYS="user1@ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCNXk5K9oRMnk70qtcO7/DaoXBMAOz29pWwIGi97Ix8Akkg5XGEKmrdYpCnknkwKeCXUqWdvRnUz3GCNc02IgSSBXrVgeGl8eM/uhcjwE45sGeBiIIi6YX4qMsVH/11puD6JOcrSD6YVbRjx+q8/ArBHtZagYngKChJNDE4QN4sjQ==;user2@ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQCNXk5K9oRMnk70qtcO7/DaoXBMAOz29pWwIGi97Ix8Akkg5XGEKmrdYpCnknkwKeCXUqWdvRnUz3GCNc02IgSSBXrVgeGl8eM/uhcjwE45sGeBiIIi6YX4qMsVH/11puD6JOcrSD6YVbRjx+q8/ArBHtZagYngKChJNDE4QN4sjQ==" \
    -e USERS="user1:password_1:::your_directory_1:s3_bucket_name_1;user2:password_2:::your_directory_2:s3_bucket_name_2" \
    --rm --cap-add SYS_ADMIN \
    --device /dev/fuse -d selfmanage-sftp:v1
```


## Advance run with Amazon ecs

coming soon