					----------/*TCP Echo Client Program*/--------------
-----/*client*/----------

#include<sys/types.h>
#include<sys/socket.h>
#include<netinet/in.h>
#include<arpa/inet.h>
#include<string.h>
#include<stdlib.h>
#include<stdio.h>

struct sockaddr_in serv_addr;
                        
int skfd, r, w;

unsigned short serv_port = 25020; /*port number used by the server*/
char serv_ip[] = "127.0.0.1"; /*server's IP-address*/
               
char rbuff[128]; /*buffer for receiving messages*/
char sbuff[128] = "===good marning==="; /*buffer for sending messages*/

int main()
{

	/*initializing server socket address structure with zero values*/
	bzero(&serv_addr, sizeof(serv_addr));

        /*filling up the server socket address structure with appropriate values*/
        serv_addr.sin_family = AF_INET; /*address family*/
        serv_addr.sin_port = htons(serv_port); /*port number*/
        inet_aton(serv_ip, (&serv_addr.sin_addr)); /*IP-address*/
	
	printf("\nTCP ECHO CLIENT.\n");

        /*creating socket*/
        if((skfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
        {
                printf("\nCLIENT ERROR: Cannot create socket.\n");
                exit(1);
        }

	/*request server for a connection*/
	if((connect(skfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr))) < 0)
	{
		printf("\nCLIENT ERROR: Cannot connect to the server.\n");
		close(skfd);
		exit(1);
	}
	printf("\nCLIENT: Connected to the server.\n");

	/*send a message to the echo server*/
	if((w = write(skfd, sbuff, 128)) < 0)       	
	{
		printf("\nCLIENT ERROR: Cannot send message to the echo server.\n");
		close(skfd);
		exit(1);
	}
	printf("\nCLIENT: Message sent to echo server.\n");

	/*read back the echoed message from server*/
	if((r = read(skfd, rbuff, 128)) < 0)
		printf("\nCLIENT ERROR: Cannot receive mesage from server.\n");
	else
	{
		rbuff[r] = '\0';

		 /*print the received message on console*/
		printf("\nCLIENT: Message from echo server: %s\n", rbuff);

	}
	close(skfd);
	exit(1);
} /*main ends*/


--------------/*Server*/--------------

#include<sys/socket.h>
#include<sys/types.h>
#include<netinet/in.h>
#include<arpa/inet.h>
#include<string.h>
#include<stdio.h>
#include<stdlib.h>

struct sockaddr_in serv_addr, cli_addr;

int listenfd, connfd, r, w, val, cli_addr_len;

unsigned short serv_port = 25020; /*port number to be used by the server*/
char serv_ip[] = "127.0.0.1"; /*server's IP-address*/

char buff[128]; /*buffer for sending and receiving messages*/

int main()
{
        /*initializing server socket address structure with zero values*/
        bzero(&serv_addr, sizeof(serv_addr));
                                                                                                                            
        /*filling up the server socket address structure with appropriate values*/
        serv_addr.sin_family = AF_INET; /*address family*/
        serv_addr.sin_port = htons(serv_port); /*port number*/
        inet_aton(serv_ip, (&serv_addr.sin_addr)); /*IP-address*/

	printf("\nTCP ECHO SERVER.\n");
                                                                                                                            
        /*creating socket*/
        if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
        {
                printf("\nSERVER ERROR: Cannot create socket.\n");
                exit(1);
        }
                                                                                                                            
        /*binding server socket address structure*/
        if((bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr))) < 0)
        {
                printf("\nSERVER ERROR: Cannot bind.\n");
                close(listenfd);
                exit(1);
        }

	/*listen to client connection requests*/
	if((listen(listenfd, 5)) < 0)
	{
		printf("\nSERVER ERROR: Cannot listen.\n");
                close(listenfd);
                exit(1);
	}

	cli_addr_len = sizeof(cli_addr);
	for( ; ;)
	{
		printf("\nSERVER: Listenning for clients...Press Cntrl + c to stop echo server:\n");
		
		/*accept client connections*/	
		if((connfd = accept(listenfd, (struct sockaddr*)&cli_addr, &cli_addr_len)) < 0)
		{
			printf("\nSERVER ERROR: Cannot accept client connections.\n");
			close(listenfd);
			exit(1);
		}
		printf("\nSERVER: Connection fron client %s accepted.\n", inet_ntoa(cli_addr.sin_addr));

		/*waiting for messages from client*/
		if((r = read(connfd, buff, 128)) < 0)
	                printf("\nSERVER ERROR: Cannot receive message from client.\n");
	        else
        	{
                	buff[r] = '\0';
                                                                                                                            
			/*echo back the message received from client*/
			if((w = write(connfd, buff, 128)) < 0)
		        	printf("\nSERVER ERROR: Cannot send message to the client.\n");
			else
				printf("\nSERVER: Echoed back %s to %s.\n", buff, inet_ntoa(cli_addr.sin_addr));
		}
	} /*for ends*/
} /*main ends*/						
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
						-----------/*Time sarver */-------------
---------/* Sarver*/--------
 #include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <time.h>

#define BUFFER_SIZE 256

int main() {
    struct sockaddr_in serv_addr, cli_addr;
    int listenfd, connfd;
    socklen_t cli_addr_len;
    char buff[BUFFER_SIZE];

    unsigned short serv_port = 25022;
    char serv_ip[] = "127.0.0.1";

    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(serv_port);
    inet_aton(serv_ip, &serv_addr.sin_addr);

    printf("\nTIME SERVER STARTED.\n");

    if ((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("SERVER ERROR: Cannot create socket");
        exit(1);
    }

    if ((bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr))) < 0) {
        perror("SERVER ERROR: Cannot bind");
        close(listenfd);
        exit(1);
    }

    if ((listen(listenfd, 5)) < 0) {
        perror("SERVER ERROR: Cannot listen");
        close(listenfd);
        exit(1);
    }

    cli_addr_len = sizeof(cli_addr);
    for (;;) {
        printf("\nSERVER: Waiting for client connections...Press Ctrl+c to stop\n");

        if ((connfd = accept(listenfd, (struct sockaddr*)&cli_addr, &cli_addr_len)) < 0) {
            perror("SERVER ERROR: Cannot accept connection");
            continue;
        }
        printf("\nSERVER: Connected to client at %s.\n", inet_ntoa(cli_addr.sin_addr));

        while (1) {
            printf("Enter 'exit' to disconnect client: ");
            char prompt[BUFFER_SIZE];
            fgets(prompt, BUFFER_SIZE, stdin);
            prompt[strcspn(prompt, "\n")] = 0;
            write(connfd, prompt, strlen(prompt) + 1);

            if (strcmp(prompt, "exit") == 0) {
                printf("\nSERVER: Disconnected client manually.\n");
                break;
            }

            bzero(buff, BUFFER_SIZE);
            if (read(connfd, buff, BUFFER_SIZE) <= 0) {
                printf("\nSERVER: Client disconnected.\n");
                break;
            }

            if (strcmp(buff, "exit") == 0) {
                printf("\nSERVER: Client requested disconnection.\n");
                break;
            }

            time_t now = time(NULL);
            struct tm *tm_info = localtime(&now);
            strftime(buff, BUFFER_SIZE, "Current Time: %Y-%m-%d %H:%M:%S", tm_info);

            write(connfd, buff, strlen(buff) + 1);
            printf("\nSERVER: Sent time to client: %s\n", buff);
        }

        close(connfd);
    }
}
------------------/*Client*/--------------------
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define BUFFER_SIZE 256

int main() {
    struct sockaddr_in serv_addr;
    int sockfd;
    char buff[BUFFER_SIZE];

    unsigned short serv_port = 25022;
    char serv_ip[] = "127.0.0.1";

    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("CLIENT ERROR: Cannot create socket");
        exit(1);
    }

    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(serv_port);
    inet_aton(serv_ip, &serv_addr.sin_addr);

    if (connect(sockfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("CLIENT ERROR: Cannot connect to server");
        close(sockfd);
        exit(1);
    }

    printf("\nCLIENT: Connected to server at %s:%d\n", serv_ip, serv_port);

    for (;;) {
        bzero(buff, BUFFER_SIZE);
        if (read(sockfd, buff, BUFFER_SIZE) <= 0) break;

        printf("\nCLIENT: Server sent: %s\n", buff);

        if (strcmp(buff, "exit") == 0) {
            printf("\nCLIENT: Server requested disconnection.\n");
            break;
        }

        printf("\nCLIENT: Press Enter to request time or type 'exit' to quit: ");
        fgets(buff, BUFFER_SIZE, stdin);
        buff[strcspn(buff, "\n")] = 0;

        if (strcmp(buff, "exit") == 0) {
            write(sockfd, buff, strlen(buff) + 1);
            break;
        }

        write(sockfd, "time", 5);

        bzero(buff, BUFFER_SIZE);
        if (read(sockfd, buff, BUFFER_SIZE) <= 0) break;

        printf("\nCLIENT: Received from server: %s\n", buff);
    }

    printf("\nCLIENT: Disconnected from server.\n");
    close(sockfd);
    return 0;
}
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
								-------/*Calculator*/------
	-----/*server*/------
 #include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

struct sockaddr_in serv_addr, cli_addr;

int listenfd, connfd, r, w;
socklen_t cli_addr_len;

unsigned short serv_port = 25022; /* Port number */
char serv_ip[] = "127.0.0.1";     /* Server IP */

char buff[128]; /* Buffer for messages */

void calculateResult(char *buffer, char *resultBuffer) {
    double num1, num2, result;
    char operator;

    sscanf(buffer, "%lf %c %lf", &num1, &operator, &num2);

    switch (operator) {
        case '+': result = num1 + num2; break;
        case '-': result = num1 - num2; break;
        case '*': result = num1 * num2; break;
        case '/': 
            if (num2 != 0) 
                result = num1 / num2; 
            else {
                strcpy(resultBuffer, "Error: Division by zero");
                return;
            }
            break;
        default: 
            strcpy(resultBuffer, "Error: Invalid operator");
            return;
    }
    
    snprintf(resultBuffer, 128, "Result: %.2f", result);
}

int main() {
    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(serv_port);
    inet_aton(serv_ip, &serv_addr.sin_addr);

    printf("\nTCP CALCULATOR SERVER.\n");

    if ((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("\nSERVER ERROR: Cannot create socket.\n");
        exit(1);
    }

    if ((bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr))) < 0) {
        printf("\nSERVER ERROR: Cannot bind.\n");
        close(listenfd);
        exit(1);
    }

    if ((listen(listenfd, 5)) < 0) {
        printf("\nSERVER ERROR: Cannot listen.\n");
        close(listenfd);
        exit(1);
    }

    cli_addr_len = sizeof(cli_addr);
    for (;;) {
        printf("\nSERVER: Listening for clients... Press Ctrl + C to stop.\n");

        if ((connfd = accept(listenfd, (struct sockaddr*)&cli_addr, &cli_addr_len)) < 0) {
            printf("\nSERVER ERROR: Cannot accept client connections.\n");
            close(listenfd);
            exit(1);
        }
        printf("\nSERVER: Connection from client %s accepted.\n", inet_ntoa(cli_addr.sin_addr));

        if ((r = read(connfd, buff, 128)) < 0)
            printf("\nSERVER ERROR: Cannot receive message from client.\n");
        else {
            buff[r] = '\0';
            printf("\nSERVER: Received calculation request: %s\n", buff);

            char resultBuffer[128];
            calculateResult(buff, resultBuffer);

            if ((w = write(connfd, resultBuffer, 128)) < 0)
                printf("\nSERVER ERROR: Cannot send result to client.\n");
            else
                printf("\nSERVER: Sent result: %s\n", resultBuffer);
        }

        close(connfd);
    }
}
----------/*Client*/----------
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

struct sockaddr_in serv_addr;
int sockfd, r, w;

unsigned short serv_port = 25022; /* Server port */
char serv_ip[] = "127.0.0.1";     /* Server IP */
char buff[128];                   /* Message buffer */

int main() {
    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(serv_port);
    inet_aton(serv_ip, &serv_addr.sin_addr);

    printf("\nTCP CALCULATOR CLIENT.\n");

    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("\nCLIENT ERROR: Cannot create socket.\n");
        exit(1);
    }

    if (connect(sockfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0) {
        printf("\nCLIENT ERROR: Cannot connect to server.\n");
        close(sockfd);
        exit(1);
    }

    printf("\nCLIENT: Connected to server at %s:%d\n", serv_ip, serv_port);

    double num1, num2;
    char operator;

    printf("\nEnter calculation (e.g., 12.5 + 3.2): ");
    scanf("%lf %c %lf", &num1, &operator, &num2);

    snprintf(buff, 128, "%.2lf %c %.2lf", num1, operator, num2);

    if ((w = write(sockfd, buff, 128)) < 0) {
        printf("\nCLIENT ERROR: Cannot send message to server.\n");
        close(sockfd);
        exit(1);
    }

    bzero(buff, 128);
    if ((r = read(sockfd, buff, 128)) < 0) {
        printf("\nCLIENT ERROR: Cannot receive result from server.\n");
    } else { 
        printf("\nCLIENT: Server response: %s\n", buff);
    }

    close(sockfd);
    return 0;
}
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
									-----/*sin Calculator*/------
	 -------/*server*/--------
  #include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <math.h>
#include <ctype.h>
#include <stdbool.h>

#define STACK_SIZE 256
#define BUFFER_SIZE 256

typedef struct {
    double data[STACK_SIZE];
    int top;
} Stack;

void push(Stack *s, double value) {
    if (s->top < STACK_SIZE - 1) s->data[++s->top] = value;
}

double pop(Stack *s) {
    return (s->top >= 0) ? s->data[s->top--] : 0;
}

char peek(Stack *s) {
    return (s->top >= 0) ? s->data[s->top] : 0;
}

int precedence(char op) {
    if (op == '+' || op == '-') return 1;
    if (op == '*' || op == '/') return 2;
    return 0;
}

double applyOp(double a, double b, char op) {
    switch (op) {
        case '+': return a + b;
        case '-': return a - b;
        case '*': return a * b;
        case '/': return (b != 0) ? a / b : NAN;
    }
    return 0;
}

double evaluateExpression(char *expr) {
    Stack values = {.top = -1};
    Stack ops = {.top = -1};
    int i = 0;

    while (expr[i] != '\0') {
        if (isspace(expr[i])) {
            i++;
            continue;
        }
        if (isdigit(expr[i]) || expr[i] == '.') {
            char *end;
            double num = strtod(&expr[i], &end);
            push(&values, num);
            i += (end - &expr[i]);
        } else if (expr[i] == '(') {
            push(&ops, expr[i]);
            i++;
        } else if (expr[i] == ')') {
            while (ops.top != -1 && peek(&ops) != '(') {
                double b = pop(&values);
                double a = pop(&values);
                char op = (char)pop(&ops);
                push(&values, applyOp(a, b, op));
            }
            pop(&ops);
            i++;
        } else {
            while (ops.top != -1 && precedence(peek(&ops)) >= precedence(expr[i])) {
                double b = pop(&values);
                double a = pop(&values);
                char op = (char)pop(&ops);
                push(&values, applyOp(a, b, op));
            }
            push(&ops, expr[i]);
            i++;
        }
    }
    while (ops.top != -1) {
        double b = pop(&values);
        double a = pop(&values);
        char op = (char)pop(&ops);
        push(&values, applyOp(a, b, op));
    }
    return pop(&values);
}

int main() {
    struct sockaddr_in serv_addr, cli_addr;
    int listenfd, connfd;
    socklen_t cli_addr_len;
    char buff[BUFFER_SIZE];

    unsigned short serv_port = 25022;
    char serv_ip[] = "127.0.0.1";

    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(serv_port);
    inet_aton(serv_ip, &serv_addr.sin_addr);

    printf("\nTCP CALCULATOR SERVER STARTED.\n");

    if ((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("SERVER ERROR: Cannot create socket");
        exit(1);
    }

    if ((bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr))) < 0) {
        perror("SERVER ERROR: Cannot bind");
        close(listenfd);
        exit(1);
    }

    if ((listen(listenfd, 5)) < 0) {
        perror("SERVER ERROR: Cannot listen");
        close(listenfd);
        exit(1);
    }

    cli_addr_len = sizeof(cli_addr);
    for (;;) {
        printf("\nSERVER: Waiting for client connections... Press Ctrl + C to terminate.\n");
        if ((connfd = accept(listenfd, (struct sockaddr*)&cli_addr, &cli_addr_len)) < 0) {
            perror("SERVER ERROR: Cannot accept connection");
            continue;
        }
        printf("\nSERVER: Connected to client at %s.\n", inet_ntoa(cli_addr.sin_addr));

        while (1) {
            // Prompt client for input
            printf("Enter exit to disconnect \n");
            char prompt[BUFFER_SIZE];
            fgets(prompt, BUFFER_SIZE, stdin);
            prompt[strcspn(prompt, "\n")] = 0;
            write(connfd, prompt, strlen(prompt) + 1);

            bzero(buff, BUFFER_SIZE);
            if (read(connfd, buff, BUFFER_SIZE) <= 0) {
                printf("\nSERVER: Wants to disconnect client.\n");
                break;
            }

            // If client sends "exit", disconnect client
            if (strcmp(buff, "exit") == 0) {
                printf("\nSERVER: Client requested disconnection.\n");
                write(connfd, "exit", 5);
                break;
            }

            printf("\nSERVER: Received expression for evaluation: %s\n", buff);

            double result = evaluateExpression(buff);
            char resultBuffer[BUFFER_SIZE];
            snprintf(resultBuffer, BUFFER_SIZE, "Result: %.2f", result);

            write(connfd, resultBuffer, strlen(resultBuffer) + 1);
            printf("\nSERVER: Sent result to client: %s\n", resultBuffer);
        }

        printf("\nSERVER: Client disconnected.\n");
        close(connfd);
    }
}
--------/*Client*/---------
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define BUFFER_SIZE 256

int main() {
    struct sockaddr_in serv_addr;
    int sockfd;
    char buff[BUFFER_SIZE];

    unsigned short serv_port = 25022;
    char serv_ip[] = "127.0.0.1";

    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("CLIENT ERROR: Cannot create socket");
        exit(1);
    }

    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(serv_port);
    inet_aton(serv_ip, &serv_addr.sin_addr);

    if (connect(sockfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("CLIENT ERROR: Cannot connect to server");
        close(sockfd);
        exit(1);
    }

    printf("\nCLIENT: Connected to server at %s:%d\n", serv_ip, serv_port);

    for (;;) {
        // Receive server prompt
        bzero(buff, BUFFER_SIZE);

        if (read(sockfd, buff, BUFFER_SIZE) <= 0) {
            printf("\nCLIENT: Server closed the connection.\n");
            break;
        }

        if (strcmp(buff, "exit") == 0) {
            printf("\nCLIENT: Disconnected from server.\n");
            close(sockfd);
            exit(0);
        }

        printf("%s", buff); // Print server prompt

        // Get user input
        printf("\nCLIENT: Enter expression to continue and (exit) to quit: ");
        fgets(buff, BUFFER_SIZE, stdin);
        buff[strcspn(buff, "\n")] = 0; // Remove newline

        // Check if user entered "exit"
        if (strcmp(buff, "exit") == 0) {
            write(sockfd, buff, strlen(buff) + 1);
            break;
        }

        write(sockfd, buff, strlen(buff) + 1);

        // Receive result
        bzero(buff, BUFFER_SIZE);
        if (read(sockfd, buff, BUFFER_SIZE) <= 0) break;

        printf("\nCLIENT: Received result from server: %s\n", buff);
    }

    printf("\nCLIENT: Disconnected from server.\n");
    close(sockfd);
    return 0;
}
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
						----------/*Chat server*/-----------
----------/*server*/-----------
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/select.h>

#define BUFFER_SIZE 256

int main() {
    struct sockaddr_in serv_addr, cli_addr;
    int listenfd, connfd;
    socklen_t cli_addr_len;
    char buff[BUFFER_SIZE];

    unsigned short serv_port = 25022;
    char serv_ip[] = "127.0.0.1";

    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(serv_port);
    inet_aton(serv_ip, &serv_addr.sin_addr);

    printf("\nCHAT SERVER STARTED.\n");

    if ((listenfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("SERVER ERROR: Cannot create socket");
        exit(1);
    }

    if ((bind(listenfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr))) < 0) {
        perror("SERVER ERROR: Cannot bind");
        close(listenfd);
        exit(1);
    }

    if ((listen(listenfd, 5)) < 0) {
        perror("SERVER ERROR: Cannot listen");
        close(listenfd);
        exit(1);
    }

    cli_addr_len = sizeof(cli_addr);
    for (;;) {
        printf("\nSERVER: Waiting for client connections... Press Ctrl+C to terminate.\n");

        if ((connfd = accept(listenfd, (struct sockaddr*)&cli_addr, &cli_addr_len)) < 0) {
            perror("SERVER ERROR: Cannot accept connection");
            continue;
        }
        printf("\nSERVER: Connected to client at %s.\n", inet_ntoa(cli_addr.sin_addr));

        fd_set read_fds;
        while (1) {
            FD_ZERO(&read_fds);
            FD_SET(STDIN_FILENO, &read_fds);
            FD_SET(connfd, &read_fds);

            printf("\nSERVER: Type a message (or 'exit' to disconnect client): ");
            fflush(stdout);

            select(connfd + 1, &read_fds, NULL, NULL, NULL);

            if (FD_ISSET(STDIN_FILENO, &read_fds)) {
                bzero(buff, BUFFER_SIZE);
                fgets(buff, BUFFER_SIZE, stdin);
                buff[strcspn(buff, "\n")] = 0;
                printf("SERVER: %s\n", buff);
                write(connfd, buff, strlen(buff) + 1);

                if (strcmp(buff, "exit") == 0) {
                    printf("\nSERVER: Disconnected client manually.\n");
                    break;
                }
            }

            if (FD_ISSET(connfd, &read_fds)) {
                bzero(buff, BUFFER_SIZE);
                if (read(connfd, buff, BUFFER_SIZE) <= 0) {
                    printf("\nSERVER: Client disconnected.\n");
                    break;
                }
                printf("\nCLIENT: %s\n", buff);

                if (strcmp(buff, "exit") == 0) {
                    printf("\nSERVER: Client requested disconnection.\n");
                    break;
                }
            }
        }

        close(connfd);
    }
}
-------/*client*/---------
#include <sys/socket.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/select.h>

#define BUFFER_SIZE 256

int main() {
    struct sockaddr_in serv_addr;
    int sockfd;
    char buff[BUFFER_SIZE];

    unsigned short serv_port = 25022;
    char serv_ip[] = "127.0.0.1";

    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("CLIENT ERROR: Cannot create socket");
        exit(1);
    }

    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(serv_port);
    inet_aton(serv_ip, &serv_addr.sin_addr);

    if (connect(sockfd, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("CLIENT ERROR: Cannot connect to server");
        close(sockfd);
        exit(1);
    }

    printf("\nCLIENT: Connected to server at %s:%d\n", serv_ip, serv_port);

    fd_set read_fds;
    while (1) {
        FD_ZERO(&read_fds);
        FD_SET(STDIN_FILENO, &read_fds);
        FD_SET(sockfd, &read_fds);

        printf("\nCLIENT: Type a message (or 'exit' to disconnect): ");
        fflush(stdout);

        select(sockfd + 1, &read_fds, NULL, NULL, NULL);

        if (FD_ISSET(STDIN_FILENO, &read_fds)) {
            bzero(buff, BUFFER_SIZE);
            fgets(buff, BUFFER_SIZE, stdin);
            buff[strcspn(buff, "\n")] = 0;
            printf("CLIENT: %s\n", buff);
            write(sockfd, buff, strlen(buff) + 1);

            if (strcmp(buff, "exit") == 0) {
                break;
            }
        }

        if (FD_ISSET(sockfd, &read_fds)) {
            bzero(buff, BUFFER_SIZE);
            if (read(sockfd, buff, BUFFER_SIZE) <= 0) break;

            printf("\nSERVER: %s\n", buff);

            if (strcmp(buff, "exit") == 0) {
                printf("\nCLIENT: Server requested disconnection.\n");
                break;
            }
        }
    }

    printf("\nCLIENT: Disconnected from server.\n");
    close(sockfd);
    return 0;
}
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
						
      
