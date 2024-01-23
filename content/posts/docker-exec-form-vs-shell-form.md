---
title: 'Dockerfile Exec Form vs Shell Form'
date: 2023-11-15T10:40:54+08:00
draft: true
---

### exec form

> The exec form is parsed as a JSON array, which means that you must use double-quotes (“) around words not single-quotes (‘).
> 
> Unlike the shell form, the exec form does not invoke a command shell. This means that normal shell processing does not happen. For example, CMD [ "echo", "$HOME" ] will not do variable substitution on $HOME. 

```
ENTRYPOINT [ "echo", "$NAME" ]
```

### shell form

> In the shell form you can use a \ (backslash) to continue a single RUN instruction onto the next line.
> 
> If you want shell processing then either use the shell form or execute a shell directly, for example: CMD [ "sh", "-c", "echo $HOME" ]

```
ENTRYPOINT echo "$NAME"
```

or

```
ENTRYPOINT [ "sh", "-c", "echo $NAME" ]
```

## References:

* [Pass ARG to ENTRYPOINT](https://stackoverflow.com/questions/50920866/pass-arg-to-entrypoint)
* [How do I use Docker environment variable in ENTRYPOINT array?](https://stackoverflow.com/questions/37904682/how-do-i-use-docker-environment-variable-in-entrypoint-array)
* [Differences Between Dockerfile Instructions in Shell and Exec Form](https://stackoverflow.com/questions/42805750/differences-between-dockerfile-instructions-in-shell-and-exec-form)