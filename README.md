# tp11serveurUDP

Il existe deux principaux protocoles de couche de transport pour communiquer entre les hôtes : TCP et UDP . La création de TCP Server/Client a été discutée dans un post précédent .

Prérequis : création du serveur/client TCP

Théorie
Dans UDP, le client n’établit pas de connexion avec le serveur comme dans TCP et envoie simplement un datagramme. De même, le serveur n’a pas besoin d’accepter une connexion et attend simplement que les datagrammes arrivent. Les datagrammes à l’arrivée contiennent l’adresse de l’expéditeur que le serveur utilise pour envoyer des données au bon client.

L’ensemble du processus peut être décomposé en les étapes suivantes :

Serveur UDP :

    Créez un socket UDP.
    Liez le socket à l’adresse du serveur.
    Attendez que le paquet de datagrammes arrive du client.
    Traiter le paquet de datagrammes et envoyer une réponse au client.
    Revenez à l’étape 3.

Client UDP :

    Créez un socket UDP.
    Envoyez un message au serveur.
    Attendez que la réponse du serveur soit reçue.
    Traitez la réponse et revenez à l’étape 2, si nécessaire.
    Fermez le descripteur de socket et quittez.

Fonctions nécessaires :

int socket(int domain, int type, int protocol)
Creates an unbound socket in the specified domain.
Returns socket file descriptor.

Arguments :
domain – Spécifie le
domaine de communication ( AF_INET pour IPv4/ AF_INET6 pour IPv6 )
type – Type de socket à créer
( SOCK_STREAM pour TCP / SOCK_DGRAM pour UDP )
protocol – Protocole à utiliser par le socket.
0 signifie utiliser le protocole par défaut pour la famille d’adresses.

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen)
Assigns address to the unbound socket.

Arguments :
sockfd – Descripteur de fichier d’une socket à lier
addr – Structure dans laquelle l’adresse à lier est spécifiée
addrlen – Taille de la structure   addr

ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
const struct sockaddr *dest_addr, socklen_t addrlen)
Send a message on the socket

Arguments :
sockfd – Descripteur de fichier de la socket
buf – Tampon de l’application contenant les données à envoyer
len – Taille des drapeaux du tampon de l’application  buf – OU bit à bit des drapeaux pour modifier le comportement de la socket  dest_addr – Structure contenant l’adresse de la destination  addrlen – Taille de dest_addr structure


ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
struct sockaddr *src_addr, socklen_t *addrlen)
Receive a message from the socket.

Arguments :
sockfd – Descripteur de fichier de la socket
buf – Tampon d’application dans lequel recevoir les données
len – Taille des drapeaux du tampon d’ application  buf – OU au niveau du bit des drapeaux pour modifier le comportement de la socket  src_addr – La structure contenant l’adresse source est renvoyée  addrlen – Variable dans laquelle la taille de la structure src_addr est retournée


int close(int fd)
Close a file descriptor

Arguments:

fd – Descripteur de fichier

Dans le code ci-dessous, l’échange d’un message hello entre le serveur et le client est illustré pour illustrer le modèle.

Nom de fichier : UDPServer.c
C

// Server side implementation of UDP client-server model
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>

#define PORT     8080
#define MAXLINE 1024

// Driver code
int main() {
int sockfd;
char buffer[MAXLINE];
char *hello = "Hello from server";
struct sockaddr_in servaddr, cliaddr;

    // Creating socket file descriptor 
    if ( (sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0 ) { 
        perror("socket creation failed"); 
        exit(EXIT_FAILURE); 
    } 
        
    memset(&servaddr, 0, sizeof(servaddr)); 
    memset(&cliaddr, 0, sizeof(cliaddr)); 
        
    // Filling server information 
    servaddr.sin_family    = AF_INET; // IPv4 
    servaddr.sin_addr.s_addr = INADDR_ANY; 
    servaddr.sin_port = htons(PORT); 
        
    // Bind the socket with the server address 
    if ( bind(sockfd, (const struct sockaddr *)&servaddr,  
            sizeof(servaddr)) < 0 ) 
    { 
        perror("bind failed"); 
        exit(EXIT_FAILURE); 
    } 
        
    int len, n; 
    
    len = sizeof(cliaddr);  //len is value/result 
    
    n = recvfrom(sockfd, (char *)buffer, MAXLINE,  
                MSG_WAITALL, ( struct sockaddr *) &cliaddr, 
                &len); 
    buffer[n] = '\0'; 
    printf("Client : %s\n", buffer); 
    sendto(sockfd, (const char *)hello, strlen(hello),  
        MSG_CONFIRM, (const struct sockaddr *) &cliaddr, 
            len); 
    printf("Hello message sent.\n");  
        
    return 0; 
}

Nom du fichier : UDPClient.c
C

// Client side implementation of UDP client-server model
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>

#define PORT     8080
#define MAXLINE 1024

// Driver code
int main() {
int sockfd;
char buffer[MAXLINE];
char *hello = "Hello from client";
struct sockaddr_in     servaddr;

    // Creating socket file descriptor 
    if ( (sockfd = socket(AF_INET, SOCK_DGRAM, 0)) < 0 ) { 
        perror("socket creation failed"); 
        exit(EXIT_FAILURE); 
    } 
    
    memset(&servaddr, 0, sizeof(servaddr)); 
        
    // Filling server information 
    servaddr.sin_family = AF_INET; 
    servaddr.sin_port = htons(PORT); 
    servaddr.sin_addr.s_addr = INADDR_ANY; 
        
    int n, len; 
        
    sendto(sockfd, (const char *)hello, strlen(hello), 
        MSG_CONFIRM, (const struct sockaddr *) &servaddr,  
            sizeof(servaddr)); 
    printf("Hello message sent.\n"); 
            
    n = recvfrom(sockfd, (char *)buffer, MAXLINE,  
                MSG_WAITALL, (struct sockaddr *) &servaddr, 
                &len); 
    buffer[n] = '\0'; 
    printf("Server : %s\n", buffer); 
    
    close(sockfd); 
    return 0; 
}

Production :

$ ./server
Client : Hello from client
Hello message sent.

$ ./client
Hello message sent.
Server : Hello from server
