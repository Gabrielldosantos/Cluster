# Cluster
Criação de um Cluster do MongoDB no Docker


docker network create mongoCluster

//criação de containers//

docker run -d --rm -p 27017:27017 --name mongo10 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo10

docker run -d --rm -p 27018:27017 --name mongo20 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo20

docker run -d --rm -p 27019:27017 --name mongo30 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo30

docker run -d --rm -p 27020:27017 --name mongo40 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip localhost,mongo40



// verificar que e o primario//

docker exec -it mongo10 mongosh


//verificar o cantainer//

db.runCommand ({hello:1})

//agora vamos inicialzar usando o camando rs.initialise//

rs.initiate ({ _id: "myReplicaSet", members:[{_id:0, host: "mongo10"}, {_id:1, host: "mongo20"}, {_id:3, host: "mongo30"}, {_id:4, host: "mongo40"}]})


//sair do mongo e ir pro power shell//

exit


//testar conjuntos de replicas//

docker exec -it mongo10 mongosh --eval "rs.status()"

ele retornando resultados significa que o ccluster conseguiu fazer a inserção com essas 4 maquinas


************mongodb************


//comando paara ver si esta conectado//

rs.ismaster() .primary

//vamos fazer a inserção de alguns dados//

use corperesystem

//comandos fazendo a inserção//

db.cliente.insertOne({codigo:1, nome: "Claudio santos"});

db.cliente.insertOne({codigo:2, nome: "Janete silva"});
db.cliente.insertOne({codigo:3, nome: "Fabio abreu"});
db.cliente.insertOne({codigo:4, nome: "Cristina chagas"});
db.cliente.insertOne({codigo:5, nome: "Renato souza"});


//pra gente coferir//

db.cliente.find()


realmente ele fez a  inserção, e ele esta fazendo a replicação com os outros nós, esse e nosso nó primario.

//parada do nosso nó primario (no docker desktop)//

docker stop mongo10

vai forçãr, vai estar desaparecendo o nó 10


//verificar novamente o status do cluster// vamos precisar ver em outro nó, o nó 10 caiu e não conseguimos ver por ele.

docker exec -it mongo20 mongosh --eval "rs.status()"

//vamos tentar ver si o cluster caiu//

db.cliente.insertOne({codigo:6, nome: "Teresa Maria"});

//vamos entrar na conexão do mongo20 ja que ele e o primario//

db.cliente.insertOne({codigo:6, nome: "Teresa Maria"});


*****alterar a identidade de 17 para 18**

//entrando no mongo20, bvamos tentar dar o  inserOne novamente//

db.cliente.insertOne({codigo:6, nome: "Teresa Maria"});

foi, ele esta replincando com outro servidor.
dessa forma conseguimos confirmar a instabiilidade do nosso sistema, a parte de autadisbilidade do mongo db, mesmo que o nó caia ele ainda comnsegue fazer a conexão.

//mostar que os registros ainda estão ali//

db.cliente.find()


mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.5.0
