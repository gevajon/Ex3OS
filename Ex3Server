/*
 ============================================================================
 Name        : ex31.c
 Author      : Jonathan Geva - 304861347
 Version     :
 Copyright   : MINE ALL MINE!!!
 Description : Server side - chomp game
 ============================================================================
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/stat.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/fcntl.h>
#include <sys/sem.h>


//function: convert string to int
int convertTOint(char* s)
{
	int i=0,sum=0;
	while((s[i] >= '0') && (s[i] <= '9'))
	{
		sum = (sum*10) + (s[i]-'0');
		i++;
	}
	return sum;
}

//function: checks if there is a ',' char in string
int checkPsik(char* s)
{
	int i=1;
	while(s[i]!=',')
	{
		i=i+1;
	}
	return i;
}

int main(void) {

	int fd;
	int shm_id;
	char* sm;
	key_t srvkey = ftok("Ex31.c",'j'); // creating a key

	shm_id = shmget(srvkey, 1024, IPC_CREAT | 0666); // gets shared memory id
	sm = (char*)shmat(shm_id,NULL,0);
	if(shm_id == -1) // checks if shared memory wasnt created
	{
		perror("failed to create shared memory\n");
		exit(0);
	}

	char buffer[100] = {'\0'};
	pid_t pid1,pid2;
	int m,n;

	mkfifo("fifo_clientTOserver",0777); //opening fifo
	write(1,"opened fifo\n",sizeof("opened fifo\n"));
	write(1,"Waiting for client request\n",sizeof("waiting for client request\n"));
	fd = open("fifo_clientTOserver",O_RDONLY);
	if(fd == -1)
	{
		write(1,"failed to create fifo\n",strlen("failed to create fifo\n"));
		exit(0);
	}
	//Waiting for client 1
	int r = read(fd,&pid1,100*sizeof(char));
	while(r==0)
	{
		r = read(fd,&pid1,100*sizeof(char));
	}
	write(1,"Waiting for client request\n",sizeof("waiting for client request\n"));
	//Waiting for client 2
	r = read(fd,&pid2,100*sizeof(char));
	while(r==0)
	{
		r = read(fd,&pid2,100*sizeof(char));
	}

	close(fd);
	unlink("fifo_clientTOserver");
	//User is entering number of rows & columns in order to create the board
	write(1,"Enter number of rows\n",sizeof("Enter number of rows\n"));
	scanf("%d",&m);
	write(1,"Enter number of columns\n",sizeof("Enter number of columns\n"));
	scanf("%d",&n);

	sprintf(buffer,"p%d,%d,%d",pid1,m,n);
	strcpy(sm, buffer);

	while(1)
	{
		int row = convertTOint(sm+1);
		int psik = checkPsik(sm+1);
		int column = convertTOint(sm+psik+2);
		if ((row == m-1) && (column == n-1) && ((sm[0] == '1') || (sm[0] == '2')))
		{
			break;
		}
	}
	sleep(1);
	write(1,"GAME OVER\n",strlen("GAME OVER\n"));
	shmdt(sm);
	shmctl(shm_id,IPC_RMID,NULL);
	return EXIT_SUCCESS;
}
