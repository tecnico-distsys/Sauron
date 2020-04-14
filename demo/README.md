# Guião de Demonstração (exemplo)

## 1. Preparação do Sistema

Para testar a aplicação e todos os seus componentes, é necessário preparar um ambiente com dados para proceder à verificação dos testes.

### 1.1. Compilar o Projeto

Primeiramente, é necessário instalar as dependências necessárias para o *silo* e os clientes (*eye* e *spotter*) e compilar estes componentes.
Para isso, basta ir à diretoria *root* do projeto e correr o seguinte comando:

```
$ mvn clean install -DskipTests
```

Com este comando já é possível analisar se o projeto compila na íntegra.

### 1.2. *Silo*

Para proceder aos testes, é preciso o servidor *silo* estar a correr. 
Para isso basta ir à diretoria *silo-server* e executar:

```
$ mvn exec:java
```

Este comando vai colocar o *silo* no endereço *localhost* e na porta *8080*.

### 1.3. *Eye*

Vamos registar 3 câmeras e as respetivas observações. 
Cada câmera vai ter o seu ficheiro de entrada próprio com observações já definidas.
Para isso basta ir à diretoria *eye* e correr os seguintes comandos:

```
$ eye localhost 8080 Tagus 38.737613 -9.303164 < eye1.txt
$ eye localhost 8080 Alameda 30.303164 -10.737613 < eye2.txt
$ eye localhost 8080 Lisboa 32.737613 -15.303164 < eye3.txt
```
**Nota:** Para correr o script *eye* é necessário fazer `mvn install` e adicionar ao *PATH* ou utilizar diretamente os executáveis gerados na diretoria `target/appassembler/bin/`.

Depois de executar os comandos acima já temos o que é necessário para testar o sistema. 

## 2. Teste das Operações

Nesta secção vamos correr os comandos necessários para testar todas as operações. 
Cada subsecção é respetiva a cada operação presente no *silo*.

### 2.1. *cam_join*

Esta operação já foi testada na preparação do ambiente, no entanto ainda é necessário testar algumas restrições.

2.1.1. Teste das câmeras com nome duplicado e coordenadas diferentes.  
O servidor deve rejeitar esta operação. 
Para isso basta executar um *eye* com o seguinte comando:

```
$ eye localhost 8080 Tagus 10.0 10.0
```

2.1.2. Teste do tamanho do nome.  
O servidor deve rejeitar esta operação. 
Para isso basta executar um *eye* com o seguinte comando:

```
$ eye localhost 8080 ab 10.0 10.0
$ eye localhost 8080 abcdefghijklmnop 10.0 10.0
```

### 2.2. *cam_info*

Esta operação não tem nenhum comando específico associado e para isso é necessário ver qual o nome do comando associado a esta operação. 
Para isso precisamos instanciar um cliente *spotter*, presente na diretoria com o mesmo nome:

```
$ mvn exec:java
```

De seguida, corremos o comando *help* e, **assumindo** que o comando se chama *info* e recebe um nome, corremos os seguintes testes:

```
> help
```

2.2.1. Teste para uma câmera existente.  
O servidor deve responder com as coordenadas de localização da câmera *Tagus* (38.737613 -9.303164):

```
> info Tagus
```

2.2.2. Teste para câmera inexistente.  
O servidor deve rejeitar esta operação:

```
> info Inexistente
```

### 2.3. *report*

Esta operação já foi testada acima na preparação do ambiente.

No entanto falta testar o sucesso do comando *zzz*. 
Na preparação foi adicionada informação que permite testar este comando.
Para testar basta abrir um cliente *spotter* e correr o comando seguinte:

```
> trail car 00AA00
```

O resultado desta operação deve ser duas observações pela câmera *Tagus* com intervalo de mais ou menos 5 segundos.

### 2.4. *track*

Esta operação vai ser testada utilizando o comando *spot* com um identificador.

2.4.1. Teste com uma pessoa (deve devolver vazio):

```
> spot person 14388236
```

2.4.2. Teste com uma pessoa:

```
> spot person 123456789
person,123456789,<timestamp>,Alameda,30.303164,-10.737613
```

2.4.3. Teste com um carro:

```
> spot car 20SD21
car,20SD21,<timestamp>,Alameda,30.303164,-10.737613
```

### 2.5. *trackMatch*

Esta operação vai ser testada utilizando o comando *spot* com um fragmento de identificador.

2.5.1. Teste com uma pessoa (deve devolver vazio):

```
> spot person 143882*
```

2.5.2. Testes com uma pessoa:

```
> spot person 111*
person,111111000,<timestamp>,Tagus,38.737613,-9.303164

> spot person *000
person,111111000,<timestamp>,Tagus,38.737613,-9.303164

> spot person 111*000
person,111111000,<timestamp>,Tagus,38.737613,-9.303164
```

2.5.3. Testes com duas ou mais pessoas:

```
> spot person 123*
person,123111789,<timestamp>,Alameda,30.303164,-10.737613
person,123222789,<timestamp>,Alameda,30.303164,-10.737613
person,123456789,<timestamp>,Alameda,30.303164,-10.737613

> spot person *789
person,123111789,<timestamp>,Alameda,30.303164,-10.737613
person,123222789,<timestamp>,Alameda,30.303164,-10.737613
person,123456789,<timestamp>,Alameda,30.303164,-10.737613

> spot person 123*789
person,123111789,<timestamp>,Alameda,30.303164,-10.737613
person,123222789,<timestamp>,Alameda,30.303164,-10.737613
person,123456789,<timestamp>,Alameda,30.303164,-10.737613
```

2.5.4. Testes com um carro:

```
> spot car 00A*
car,00AA00,<timestamp>,Tagus,38.737613,-9.303164

> spot car *A00
car,00AA00,<timestamp>,Tagus,38.737613,-9.303164

> spot car 00*00
car,00AA00,<timestamp>,Tagus,38.737613,-9.303164
```

2.5.5. Testes com dois ou mais carros:

```
> spot car 20SD*
car,20SD20,<timestamp>,Alameda,30.303164,-10.737613
car,20SD21,<timestamp>,Alameda,30.303164,-10.737613
car,20SD22,<timestamp>,Alameda,30.303164,-10.737613

> spot car *XY20
car,66XY20,<timestamp>,Lisboa,32.737613,-15.303164
car,67XY20,<timestamp>,Alameda,30.303164,-10.737613
car,68XY20,<timestamp>,Tagus,38.737613,-9.303164

> spot car 19SD*9
car,19SD19,<timestamp>,Lisboa,32.737613,-15.303164
car,19SD29,<timestamp>,Lisboa,32.737613,-15.303164
car,19SD39,<timestamp>,Lisboa,32.737613,-15.303164
car,19SD49,<timestamp>,Lisboa,32.737613,-15.303164
car,19SD59,<timestamp>,Lisboa,32.737613,-15.303164
car,19SD69,<timestamp>,Lisboa,32.737613,-15.303164
car,19SD79,<timestamp>,Lisboa,32.737613,-15.303164
car,19SD89,<timestamp>,Lisboa,32.737613,-15.303164
car,19SD99,<timestamp>,Lisboa,32.737613,-15.303164
```

### 2.6. *trace*

Esta operação vai ser testada utilizando o comando *trail* com um identificador.

2.6.1. Teste com uma pessoa (deve devolver vazio):

```
> trail person 14388236
```

2.6.2. Teste com uma pessoa:

```
> trail person 123456789
person,123456789,<timestamp>,Alameda,30.303164,-10.737613
person,123456789,<timestamp>,Alameda,30.303164,-10.737613
person,123456789,<timestamp>,Tagus,38.737613,-9.303164

```

2.6.3. Teste com um carro (deve devolver vazio):

```
> trail car 12XD34
```

2.6.4. Teste com um carro:

```
> trail car 00AA00
car,00AA00,<timestamp>,Tagus,38.737613,-9.303164
car,00AA00,<timestamp>,Tagus,38.737613,-9.303164
```

## 3. Considerações Finais

Apesar de não serem avaliados os comandos de controlo, o comando *help* deve ser minimamente informativo e deve indicar todas as operações existentes no *spotter*.
Estes testes não cobrem tudo, pelo que devem ter sempre em conta os testes de integração e o código.

