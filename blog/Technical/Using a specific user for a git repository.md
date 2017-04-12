# Using a specific user for a git repository

We use git a lot. Recently I met a problem, for different git repositories, I used the differents emails and usernames. Initially, I set up my git global username and email. Now, it could not work in this situation. Here is the solution.

Actually in git, each repository contains a folder named ".git". In this folder, there is a file named "config". We can add configurations here. However, I use this way,

1) in the repository folder, using the following commands ->

```shell
git config user.email youremail.address
git config user.name "yourname"
```

2) then in this repository, the email and username is specified as above information.

It works well for me, 

```
git version 2.7.4 (Apple Git-66)
```

