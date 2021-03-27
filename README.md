# Exercício 14 - Kafka

Alteramos a aplicação para ao invés de envir um objeto json, vamos enviar um objeto serializado com [avro](https://avro.apache.org/), integramos também com o schema registry. Veja as alterações no código, principalmente o arquivo [UserChangedMessage.avsc](/sample-app/src/main/avro/UserChangedMessage.avsc) (descreve o schema da mensagem).

Suba a aplicação novamente:

```console
cd ~/hands-on-microservices/sample-app/

git checkout e14

./gradlew clean build

java -jar build/libs/sample-app-0.0.14-SNAPSHOT.jar
```

Crie alguns usuários via http://172.0.2.32:30001/swagger-ui.html, ou via curl:

```console
curl localhost:30001/users/random
```

Verifique a aba schema do tópico no control center, veja que temos um schema registrado para esse tópico. Mude a compatibilidade para FORWARD (ou seja, o antigo schema deve poder ler o que for escrito no novo schema).

Altere então o schema [UserChangedMessage.avsc](/sample-app/src/main/avro/UserChangedMessage.avsc) do avro da aplicação, deixando o nome opcional: 

```console
vim ~/hands-on-microservices/sample-app/src/main/avro/UserChangedMessage.avsc
```

Altere o atributo userName para isso:

```avro
...
{
  "name": "userName",
  "type": [
    "null",
    "string"
  ]
} 
```

Recompile, reinicie a aplicação e crie um novo usuário. Você devera ver o seguine erro no log e o usuário não vai ser criado (a API vai retornar erro 500).

```console
org.apache.kafka.common.errors.SerializationException: Error registering Avro schema: {"type":"record","name":"UserChangedMessage","namespace":"web.core.user.pub","fields":[{"name":"userId","type":{"type":"string","avro.java.string":"String"}},{"name":"userName","type":["null",{"type":"string","avro.java.string":"String"}]}]}
Caused by: io.confluent.kafka.schemaregistry.client.rest.exceptions.RestClientException: Schema being registered is incompatible with an earlier schema for subject
```

Porque ocorreu o erro? Como podemos corrigir? 

Altere a compatibilidade do schema no control center para NONE, tente gerar um novo usuário agora. Agora funcionou certo? mas por que essa deve ser a última opção para resolver esse problema?

Agora crie um usuário com email do **hotmail**, veja no log da aplicação o que ocorreu. Esse é um problema de consistência, como resolver?

```console
curl localhost:30001/users/random?emailDomain=hotmail
```
