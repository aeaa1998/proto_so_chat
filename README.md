# Proto uso

## Llenado del Payload

Todos los payloads se les pone esto siempre client -> server.<br>
``` c++
payload.set_sender(mi_username);
payload.set_ip(mi_ip);
```

Todos los payloads se les pone esto siempre server -> client.<br>
```c++
server_payload.set_sender("server");
//200 o 500
int code = 200;

server_payload.set_code(code);
//Aca metemos la flag original por si el client desea llevar un registro o algo
server_payload.set_flag(payload.flag());
```

### Mensaje privado
Flag -> private_chat<br>
Extra -> username del que recibe<br>
Message -> El mensaje<br>
``` c++
payload.set_flag(Payload_PayloadFlag::Payload_PayloadFlag_private_chat)
//El extra se llena con el username del recipient
payload.set_extra(username_recipient)
payload.set_message(message)
//Se serializa y se manda al server
```

### Mensaje general
Flag -> general_chat<br>
Message -> El mensaje<br>
``` c++
payload.set_flag(Payload_PayloadFlag::Payload_PayloadFlag_general_chat)
payload.set_message(message)
//Se serializa y se manda al server
```

### Cambio de status
Flag -> update_status<br>
Extra -> El status deseado<br>
``` c++
payload.set_flag(Payload_PayloadFlag::Payload_PayloadFlag_update_status)
string status = 'INACTIVO'; //PUEDE SER ACITVO, INACTIVO u OCUPADO
payload.set_extra(status)
//Se serializa y se manda al server
```

### Solicitar información de un usuario
Flag -> user_info<br>
Extra -> El username del usuario deseado<br>
``` c++
payload.set_flag(Payload_PayloadFlag::Payload_PayloadFlag_user_info)
payload.set_extra(desired_username)
//Se serializa y se manda al server
```

### Solicitar lista de usuarios
Solo necesita saber que se desea la lista de usuarios unicamente se necesita el flag<br>
Flag -> user_list<br>
``` c++
payload.set_flag(Payload_PayloadFlag::Payload_PayloadFlag_user_list)
//Se serializa y se manda al server
```

### Registro
La ip y el sender siempre van en el codigo del client solo se necesita el flag.<br>
Flag -> register<br>
``` c++
payload.set_flag(Payload_PayloadFlag::Payload_PayloadFlag_register_)
//Se serializa y se manda al server
```

## Ejemplos
#### Server
Ejemplo de como usar el proto para mandar un mensaje privado<br>
``` c++
    if (payload.flag() == Payload_PayloadFlag::Payload_PayloadFlag_private_chat)
    {
        for (int i = 0; i < MAX_CLIENTS; ++i)
        {
            if (clients[i])
            {
                if (strcmp(clients[i]->name, payload.extra().c_str()) == 0)
                {
                    //Se manda solo el string del mensaje privado al username destinado
                    //El client ya solo imprime el texto sin tener que pensar que respuesta es
                    //MASK: {sender_username} + (privado): + {mensaje}
                    string message_priv = payload.sender() + " (private): " + payload.message();
                    Payload server_payload;
                    server_payload.set_sender("server");
                    server_payload.set_message(message_priv);
                    server_payload.set_code(200);
                    string out;
                    server_payload.SerializeToString(&out);
                    if (write(clients[i]->socket_d, out.c_str(), out.length()) < 0)
                    {
                        //Aca se mandaria 500 con el mensaje de error solo como ejemplo
                        perror("Error: no se pudo enviar el mensaje privado");
                        break;
                    }
                    break;
                }
            }
        }
    }
```

#### Client
Ejemplo de como recibir e imprimir el payload sin importar de que tipo sea en el client.<br>
``` c++
        int received_message = recv(sockfd, message, LENGTH, 0);
        if (received_message > 0)
        {
            Payload server_payload;
            server_payload.ParseFromString(message);
            //En caso sea un mensaje general o sea un status code de 200
            if (server_payload.code() == 200 || server_payload.flag() == Payload_PayloadFlag::Payload_PayloadFlag_general_chat)
            {
                printf("%s \n", server_payload.message().c_str());
            }
            else
            {
                printf("Error del server %d -- %s!", server_payload.code(), server_payload.message().c_str())
                //Por ejemplo aca un output podria ser
                //"Error del server 500 -- El username que ingreso para mandarle un mensaje privado no existe."
            }
        }
````
