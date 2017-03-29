/*
 ============================================================================
 Name        : ex32.c
 Author      : Jonathan Geva - 304861347
 Version     :
 Copyright   : MINE ALL MINE!!
 Description : Client side - chomp game
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

//function: converts string to int
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
	while(s[i] != ',')
	{
		i=i+1;
	}
	return i;
}

int main(void) {

	//FIFO - Initialization
	int fd = open("fifo_clientTOserver",O_WRONLY);
	pid_t pid = getpid();
	write(fd,&pid,sizeof(pid_t));
	close(fd);

	char player = '\0';
	char buffer[100];
	int m,n,psik,nl;

	//Shared memory - definition
	key_t srvpid = ftok("Ex31.c",'j');
	int shm_id;
	char* sm;
	shm_id = shmget(srvpid, 1024 , 0666);
	if(shm_id == -1)
	{
		perror("failed to create shared memory\n");
	}
	sm = (char*)shmat(shm_id,NULL,0);
	strcpy(sm,buffer);

	//in order to wait for the first pid
	while(buffer[0] != 'p')
	{
		strcpy(buffer,sm);
	}
	//write (1,buffer,strlen(buffer));
	int turn = 0;
	srvpid = convertTOint(buffer+1);
	if(srvpid == pid)
	{
		player = '1';
		turn = 1;
	}
	else
	{
		player = '2';
	}

	psik = checkPsik(buffer);
	m = convertTOint(buffer+psik+1);
	nl = checkPsik(buffer+psik);
	n = convertTOint(buffer+psik+nl+1);

	char* place = (char*)malloc(sizeof(char)*m*n); // creating board dynamically
	int i=0,j=0;
	for(i=0;i<m;i++) // printing the board with 'X' in each cell (clean board)
	{
		for(j=0;j<n;j++)
		{
			place[i*n+j] = 'X';
		}
	}

	//printing the board
	for(i=0;i<m;i++)
	{
		for(j=0;j<n;j++)
		{
			write(1,&place[i*n+j],sizeof(char));
		}
		write(1,"\n",sizeof(char));
	}

	int choose=-1;
	int row=-1,column=-1;
	char x;

	while(1)
	{
		if (turn)
		{
			//its your turn - make a move
			choose = -1;
			while(choose==-1)
			{
				write(1,"Please choose a row and a column\n", sizeof("Please choose a row and a column\n"));
				scanf("%d%c%d",&row,&x,&column);
				if((row > m-1) || (column > n-1)) // if player choose point (X,Y) out of game board
				{
					write(1,"No such place\n",strlen("No such place\n"));
					choose = -1;
				}
				else
				{
					if (place[row*(n)+column] != 'X') // if the current cell is not an 'X' - its empty already
					{
						write(1,"This place is empty\n",strlen("This place is empty\n"));
						choose = -1;
					}
					else
					{
						choose = 0;
					}
				}
			}
			if (player == '1')
				sprintf(buffer,"%c%d,%d",'2',row,column);
			else
				sprintf(buffer,"%c%d,%d",'1',row,column);
			//updating the board - from point [row,column] left and up = ' ' (empty cell)
			for(i=0;i<=row;i++)
			{
				for(j=0;j<=column;j++)
				{
					place[i*n+j] = ' ';
				}
			}

			//printing the board
			for(i=0;i<m;i++)
			{
				for(j=0;j<n;j++)
				{
					write(1,&place[i*n+j],sizeof(char));
				}
				write(1,"\n",sizeof(char));
			}
			strcpy(sm, buffer);
			sleep(1);
			if (row == m-1 && column == n-1)
				break;
			turn = 0;
		}
		else	//Not player's turn - Waiting session
		{
			write(1,"Waiting for the other player to make a move\n",strlen("Waiting for the other player to make a move\n"));
			strcpy(buffer,sm);
			while(buffer[0] != player) // in order to wait for turn
			{
				sleep(1);
				strcpy(buffer,sm);
			}
			row = convertTOint(buffer+1);
			psik = checkPsik(buffer+1);
			column = convertTOint(buffer+2+psik);

			//updating the board - from point [row,column] left and up = ' ' (empty cell)
			for(i=0;i<=row;i++)
			{
				for(j=0;j<=column;j++)
				{
					place[i*n+j] = ' ';
				}
			}

			//printing the board
			for(i=0;i<m;i++)
			{
				for(j=0;j<n;j++)
				{
					write(1,&place[i*n+j],sizeof(char));
				}
				write(1,"\n",sizeof(char));
			}
			if (row == m-1 && column == n-1)
				break;
			turn = 1;

		}
	}
	shmdt(sm);
	if (turn)
		write(1,"You lost!\n",strlen("You lost!\n")); //if its your turn and you chose the (m,n) point -> you lost
	else
		write(1,"You win!\n",strlen("You win!\n")); // the other player won due to the current player has to choose that point

	return EXIT_SUCCESS;
}
