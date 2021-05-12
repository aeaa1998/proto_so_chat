# Proto uso
## Server
Ejemplo de como usar el proto para mandar un mensaje privado
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
                    Payload payload;
                    payload.set_sender("server");
                    payload.set_message(message_priv);
                    payload.set_code(200);
                    string out;
                    payload.SerializeToString(&out);
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

## Client
Ejemplo de como recibir e imprimir el payload sin importar de que tipo sea en el client.
``` c++
        int received_message = recv(sockfd, message, LENGTH, 0);
        if (received_message > 0)
        {
            Payload server_payload;
            server_payload.ParseFromString(message);
            if (server_payload.code() == 200)
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