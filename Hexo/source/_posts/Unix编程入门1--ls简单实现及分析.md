title: Unix编程入门1--ls简单实现及分析
date: 2014-09-20 09:05:24
tags: [Code,Unix,Linux]
categories: Unix
---
这个入门系列来自《UNIX高级编程》之后的东西大都是来自这本书，在这里自己记录下。

---

```c
#include "apue.h"
#include <dirent.h>
 
int main(int argc, char* argv[])
{
	DIR *dp;
	struct dirent *dirp;
 
	if (argc != 2)
		err_quit("usage: ls directioy_name");
 
	if((dp = opendir(argv[1])) == NULL)
		err_sys("can't open %s", argv[1]);
 
	while ((dirp = readdir(dp)) != NULL)
		printf("%s\n", dirp->d_name);
 
	closedir(dp);
	exit(0);
}
```

上面这段就是*ls*指令的一个简单的实现，当然如果你直接复制这段代码的话它是无法编译通过的。书中将一些常用的头文件以及异常的处理都放在了一起，也就是*apue.h*中所以在这段程序编译之前我们需要进行以下工作。

首先进入[这本书的官网](http://www.apuebook.com/),打开最新版的链接下载Source Code中的源码，下载完成后，解压文件在找到路径下的./apue.3e/include/apue.h和./apue.3e/lib/error.c将其移动到/usr/include下，（sudo cp 啥的= =都知道吧）之后打开apue.h将error.c加入apue.h中，具体位置实在结尾处的#endif之前
这样：

apue.h

```c
/* Our own header, to be included before all standard system headers.*/
 
#ifndef	_APUE_H
#define	_APUE_H
 
#define _POSIX_C_SOURCE 200809L
 
#if defined(SOLARIS)		/* Solaris 10 */
#define _XOPEN_SOURCE 600
#else
#define _XOPEN_SOURCE 700
#endif
 
#include <sys/types.h>		/* some systems still require this */
#include <sys/stat.h>
#include <sys/termios.h>	/* for winsize */
#if defined(MACOS) || !defined(TIOCGWINSZ)
#include <sys/ioctl.h>
#endif
 
#include <stdio.h>		/* for convenience */
#include <stdlib.h>		/* for convenience */
#include <stddef.h>		/* for offsetof */
#include <string.h>		/* for convenience */
#include <unistd.h>		/* for convenience */
#include <signal.h>		/* for SIG_ERR */
 
#define	MAXLINE	4096			/* max line length */
 
/* Default file access permissions for new files. */
 
#define	FILE_MODE	(S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH)
 
/* Default permissions for new directories. */
#define	DIR_MODE	(FILE_MODE | S_IXUSR | S_IXGRP | S_IXOTH)
 
typedef	void	Sigfunc(int);	/* for signal handlers */
 
#define	min(a,b)	((a) < (b) ? (a) : (b))
#define	max(a,b)	((a) > (b) ? (a) : (b))
 
/* Prototypes for our own functions. */
 
char	*path_alloc(size_t *);				/* {Prog pathalloc} */
long	 open_max(void);					/* {Prog openmax} */
 
int		 set_cloexec(int);					/* {Prog setfd} */
void	 clr_fl(int, int);
void	 set_fl(int, int);					/* {Prog setfl} */
 
void	 pr_exit(int);						/* {Prog prexit} */
 
void	 pr_mask(const char *);				/* {Prog prmask} */
Sigfunc	*signal_intr(int, Sigfunc *);		/* {Prog signal_intr_function} */
 
void	 daemonize(const char *);			/* {Prog daemoninit} */
 
void	 sleep_us(unsigned int);			/* {Ex sleepus} */
ssize_t	 readn(int, void *, size_t);		/* {Prog readn_writen} */
ssize_t	 writen(int, const void *, size_t);	/* {Prog readn_writen} */
 
int		 fd_pipe(int *);					/* {Prog sock_fdpipe} */
int		 recv_fd(int, ssize_t (*func)(int,
		         const void *, size_t));	/* {Prog recvfd_sockets} */
int		 send_fd(int, int);					/* {Prog sendfd_sockets} */
int		 send_err(int, int,
		          const char *);			/* {Prog senderr} */
int		 serv_listen(const char *);			/* {Prog servlisten_sockets} */
int		 serv_accept(int, uid_t *);			/* {Prog servaccept_sockets} */
int		 cli_conn(const char *);			/* {Prog cliconn_sockets} */
int		 buf_args(char *, int (*func)(int,
		          char **));				/* {Prog bufargs} */
 
int		 tty_cbreak(int);					/* {Prog raw} */
int		 tty_raw(int);						/* {Prog raw} */
int		 tty_reset(int);					/* {Prog raw} */
void	 tty_atexit(void);					/* {Prog raw} */
struct termios	*tty_termios(void);			/* {Prog raw} */
 
int		 ptym_open(char *, int);			/* {Prog ptyopen} */
int		 ptys_open(char *);					/* {Prog ptyopen} */
#ifdef	TIOCGWINSZ
pid_t	 pty_fork(int *, char *, int, const struct termios *,
		          const struct winsize *);	/* {Prog ptyfork} */
#endif
 
int		lock_reg(int, int, int, off_t, int, off_t); /* {Prog lockreg} */
 
#define	read_lock(fd, offset, whence, len) \
			lock_reg((fd), F_SETLK, F_RDLCK, (offset), (whence), (len))
#define	readw_lock(fd, offset, whence, len) \
			lock_reg((fd), F_SETLKW, F_RDLCK, (offset), (whence), (len))
#define	write_lock(fd, offset, whence, len) \
			lock_reg((fd), F_SETLK, F_WRLCK, (offset), (whence), (len))
#define	writew_lock(fd, offset, whence, len) \
			lock_reg((fd), F_SETLKW, F_WRLCK, (offset), (whence), (len))
#define	un_lock(fd, offset, whence, len) \
			lock_reg((fd), F_SETLK, F_UNLCK, (offset), (whence), (len))
 
pid_t	lock_test(int, int, off_t, int, off_t);		/* {Prog locktest} */
 
#define	is_read_lockable(fd, offset, whence, len) \
			(lock_test((fd), F_RDLCK, (offset), (whence), (len)) == 0)
#define	is_write_lockable(fd, offset, whence, len) \
			(lock_test((fd), F_WRLCK, (offset), (whence), (len)) == 0)
 
void	err_msg(const char *, ...);			/* {App misc_source} */
void	err_dump(const char *, ...) __attribute__((noreturn));
void	err_quit(const char *, ...) __attribute__((noreturn));
void	err_cont(int, const char *, ...);
void	err_exit(int, const char *, ...) __attribute__((noreturn));
void	err_ret(const char *, ...);
void	err_sys(const char *, ...) __attribute__((noreturn));
 
void	log_msg(const char *, ...);			/* {App misc_source} */
void	log_open(const char *, int, int);
void	log_quit(const char *, ...) __attribute__((noreturn));
void	log_ret(const char *, ...);
void	log_sys(const char *, ...) __attribute__((noreturn));
void	log_exit(int, const char *, ...) __attribute__((noreturn));
 
void	TELL_WAIT(void);		/* parent/child from {Sec race_conditions} */
void	TELL_PARENT(pid_t);
void	TELL_CHILD(pid_t);
void	WAIT_PARENT(void);
void	WAIT_CHILD(void);
 
 
#inlcude "error.c"  /*加到这里*/
 
#endif	/* _APUE_H */
```

以上的工作完成后我们就可以编译运行这个程序了

我们将可执行文件命名为myls，当权限允许时，执行myls后会得到作为参数输入路径的所用文件的文件名

* ./myls /

这样程序就会返回root下的所有文件名

下面介绍下这段程序中的API：

```c
#include "apue.h"
#include <dirent.h>
 
int main(int argc, char* argv[])
{
	DIR *dp;
	struct dirent *dirp;
 
	if (argc != 2)
		err_quit("usage: ls directioy_name");
 
	if((dp = opendir(argv[1])) == NULL)
		err_sys("can't open %s", argv[1]);
 
	while ((dirp = readdir(dp)) != NULL)
		printf("%s\n", dirp->d_name);
 
	closedir(dp);
	exit(0);
}
```
```c
DIR *opendir(const char *pathname);
//成功时返回DIR指针，出错返回NULL
//获取DIR指针
 
struct dirent *readdir(DIR *dp);
//成功返回指针，若在目录尾出错返回NULL
//返回值为目录中的第一个目录项（个人感觉类似文件读取时的文件指针FILE*）
//例子中当执行到文件尾端是返回NULL这样可以便利本目录下的所有目录

int closedir(DIR *dp);
//成功返回0，出错返回-1
//销毁指针
 
//DIR是一种目录处理需要的结构，保存目录相关信息，由打开文件的描述符转化而来
//struct dir中记录的为文件信息，显示为以下形式
struct dirent
{
	long d_ino; /* inode number 索引节点号 */
	off_t d_off; /* offset to this dirent 在目录文件中的偏移 */
	unsigned short d_reclen; /* length of this d_name 文件名长 */
	unsigned char d_type; /* the type of d_name 文件类型 */
	char d_name [NAME_MAX+1]; /* file name (null-terminated) 文件名，最长256字符 */
}
```

以上

---

如果什么地方有错误的话请联系我~
这是我的邮箱：albstein2@gmail.com