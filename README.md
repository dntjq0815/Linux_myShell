# Linux_myShell
 Creating my own Linux shell with C





#### Implemented Commands
 
 Command | Description
 ---| ---|
 exit | close the shell
 pwd | print working directory
 cat | reads files sequentially, writing them to standard output
 cd | change the working directory
 cp | copy the file
 ls | print lists of existing files in direcotry
 mkdir | make directories
 rm | remove files or directories
 stat | print the status of the file
 chmod | change the access code (authority)
 ln | create a hard link
 ps | print working processes
 
 ## pwd
 
 ```c
 void pwd() {
	char cwd[255];
	getcwd(cwd, sizeof(cwd));
	printf("%s ", cwd);
 }
 ```
 
 
 ## cat
 
 ```c
 int cat(int argc, char *argv[]) {
	for (int i = 1; i < argc; i++) {
		FILE *fp;
	        char buf[1000];
		
		fp = fopen(argv[i], "r");

		if (!fp) 
			perror(argv[i]);
		while (fgets(buf, 1000, fp) != NULL)
			printf("%s", buf);
		fclose(fp);
	}	
	return 0;
}
```

## cd

```c
int cd(int argc, char **argv) {
	if (argc != 2) {
		fprintf(stderr, "Usage: %s <path name>\n", argv[0]);
		return 1;
	}
	
	if(chdir(argv[1]) == -1) 
		printf("Failed! Check the directory\n");
	
	return 0;
}
```

## cp
```c
int cp(int argc, char*argv[]) {
	char *cmd1;
	char *cmd2;
	char block[MAXSIZE];
	int in, out;
	int nread;
	
	if (argc != 3) {
		printf("%s [file name] [copy file name]\n", argv[0]);
		exit(0);
	}
	
	cmd1 = argv[1];
	cmd2 = argv[2];
	
	in = open(cmd1, O_RDONLY);
	out = open(cmd2, O_WRONLY| O_CREAT, S_IRUSR| S_IWUSR);
	nread = read(in, block, sizeof(block));
	write(out, block, nread);
}
```

## ls
```c
int ls() {
	char* cwd = (char*)malloc(sizeof(char)*1024);
	DIR* dir = NULL;
	struct dirent* entry = NULL;
	
	getcwd(cwd, 1024);
	
	if ((dir = opendir(cwd)) == NULL) {
		printf("current directory error\n");
	}
	
	while ((entry = readdir(dir)) != NULL) {
		printf("%s\n", entry->d_name);
	}
	
	free(cwd);
	closedir(dir);
	
	return 0;
}
```

## mkdir
```c
int md(char **argv, int mode) {
	if(mkdir(argv[1], mode) == -1) 
		printf("Failed! Already exists\n");
	
	return 0;
}
```

## rm
```c
int rm(int argc, char **argv) {
	if(rmdir(argv[1]) == -1) 
		printf("Failed! In-file exists\n");
	else
		remove(argv[1]);
	return 0;
}
```

## stat
```c
int myStat(int argc, char *argv[]) {
	struct stat st;
	
	if (argc != 2) {
		fprintf(stderr, "Usage: %s <pathname>\n", argv[0]);
		return 1;
	}
	
	if (stat(argv[1], &st) == -1) {
		perror("stat");
		return 1;
	}
	
	printf("File type: "); 
	switch (st.st_mode & S_IFMT) { 
		case S_IFBLK: printf("block device\n"); break; 
		case S_IFCHR: printf("character device\n"); break; 
		case S_IFDIR: printf("directory\n"); break; 
		case S_IFIFO: printf("FIFO/pipe\n"); break;  
		case S_IFREG: printf("regular file\n"); break; 
		default: printf("unknown?\n"); break; 
	} 
	
	printf("I-node number: %ld\n", (long) st.st_ino); 
	printf("Mode: %lo (octal)\n", (unsigned long) st.st_mode); 
	printf("Link count: %ld\n", (long) st.st_nlink); 
	printf("Ownership: UID=%ld GID=%ld\n", (long) st.st_uid, (long) st.st_gid); 
	printf("File size: %lld bytes\n", (long long) st.st_size); 
	printf("Last status change: %s", ctime(&st.st_ctime)); 
	printf("Last file access: %s", ctime(&st.st_atime)); 
	printf("Last file modification: %s", ctime(&st.st_mtime)); 
	
	return 0;
}
```

## chmod
```c
int cm(char *argv[], int mode) {
	if (chmod(argv[1], mode) == -1)
		printf("ERROR!\n");
	
	else if (chmod(argv[1], mode) == 0)
		printf("Successfully Changed\n");
		
	return 0;
}
```

## ln
```c
int ln(char *argv[], char *argv2[]) {
	if (link(argv[1], argv[2]) == -1)
        	printf("error");
	
	return 0;
}
```

## ps
```c
int ps(int argc, char *argv[]) {
	DIR *dir;
  	struct dirent *ent;
  	int i, fd_self, fd;
  	unsigned long time, stime;
  	char flag, *tty;
  	char cmd[256], tty_self[256], path[300], time_s[256];
  	FILE* file;

  	dir = opendir("/proc");
  	fd_self = open("/proc/self/fd/0", O_RDONLY);
  	sprintf(tty_self, "%s", ttyname(fd_self));
  	printf(FORMAT, "PID", "TTY", "TIME", "CMD");

  	while ((ent = readdir(dir)) != NULL)
  	{
  		flag = 1;
  		for (i = 0; ent->d_name[i]; i++)
  		if (!isdigit(ent->d_name[i]))
  		{ 
   			flag = 0;
   			break;
  		}

  		if (flag)
  		{
  			sprintf(path, "/proc/%s/fd/0", ent->d_name);
  			fd = open(path, O_RDONLY);
  			tty = ttyname(fd);

  			if (tty && strcmp(tty, tty_self) == 0)
  			{

   				sprintf(path, "/proc/%s/stat", ent->d_name);
   				file = fopen(path, "r");
   				fscanf(file, "%d%s%c%c%c", &i, cmd, &flag, &flag, &flag);
   				cmd[strlen(cmd) - 1] = '\0';

  				for (i = 0; i < 11; i++)
  				fscanf(file, "%lu", &time);
  				fscanf(file, "%lu", &stime);
  				time = (int)((double)(time + stime) / sysconf(_SC_CLK_TCK));
  				sprintf(time_s, "%02lu:%02lu:%02lu",
  				(time / 3600) % 3600, (time / 60) % 60, time % 60);
 
  				printf(FORMAT, ent->d_name, tty + 5, time_s, cmd + 1);
  				fclose(file);
  			}
 			close(fd);
		}
	}
	close(fd_self);
	return 0;	
}
```

## parsing
```c
int parseLine(char **argv, char *line) {
	char *ptr;
	char token[] = " \n";
	
	if (fgets(line, MAXLINE, stdin) == NULL) {
      printf("ERROR");
    }
	ptr = strtok(line, token);
	
	int idx = 0;
	while(ptr != NULL) {
		argv[idx] = ptr;
		ptr = strtok(NULL, token);	
		idx += 1;
	}
	return idx;
}
```
parseLine() is a function which parses the line that user entered on the shell. <br/>
**For example**, when **"cd test"** is entered, argv[0] refers to 'cd', and argv[1] refers to 'test'. <br/>

## main()
```c
int main() {
	char line[MAXLINE];
	char *argv[MAXSIZE] = {""};
   
        while(strcmp(argv[0], "exit") != 0) {
    		print_prompt();
    		int argc = parseLine(argv, line);
    	
    		if(strcmp(argv[0], "pwd") == 0) {
    			pwd();
	    		printf("\n\n");
		}
		else if(strcmp(argv[0], "cat") == 0) {
			cat(argc, argv);
			printf("\n");
		}
		else if(strcmp(argv[0], "cd") == 0) {
			cd(argc, argv);
			printf("\n");
		}
		else if(strcmp(argv[0], "cp") == 0) {
			cp(argc, argv);
			printf("\n");
		}
		else if(strcmp(argv[0], "ls") == 0) {
			ls();
			printf("\n");
		}
		else if(strcmp(argv[0], "mkdir") == 0) {
			md(argv, 0777);
			printf("\n");
		}
		else if(strcmp(argv[0], "rm") == 0) {
			rm(argc, argv);
			remove(argv[1]);
			printf("\n");
		}
		else if(strcmp(argv[0], "stat") == 0) {
			myStat(argc, argv);
			printf("\n");
		}
		else if(strcmp(argv[0], "chmod") == 0) {
			int mode = atoi(argv[2]);
			cm(argv, mode);
			printf("\n");
		}
		else if (strcmp(argv[0], "ln") == 0) {
			ln(argv, argv);
			printf("\n");
		}
		else if (strcmp(argv[0], "ps") == 0) {
			ps(argc, argv);
			printf("\n");
		}
       	}
}
```
From just right above, we can distinguish the commands by comparing only 'argv[0]'. <br/>
To check which command did the user entered, implemented function, strcmp() is needed.
