# Basic Shell Script to Create User

## Requirement
- Linux

## Step

1. Create script with your desired name, example `create-user.sh`, then copy content below

```
#!/bin/bash

## Created by Immanuelbint

## Function read user input
read_user_input() {
    read -r -p "Enter username you'd like to create = " username
}

## Function create user
create_user() {
    sudo useradd "$username"
}

## Function check if user exists
check_if_user_exist() {
    read_user_input
    while grep -q "$username" /etc/passwd;
    do
        echo "User $username already exist's, please enter other name"
        read_user_input
    done
    create_user && echo "create user $username success"
}

## Main program
check_if_user_exist
```

2. Set the script executable with command 

```
chmod +x <namescript.sh>
```

3. Then running the script without <>

```
bash <namescript.sh>
```