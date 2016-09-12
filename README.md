# base_project_docker
Projeto base, utilizando docker com uma imagem ubuntu;mysql;wordpress


Instalar docker
-------------------------------
Rodar docker automaticamente ao iniciar ubuntu
```sh
	$ sudo apt-get -y install docker.io
```
Rodar serviço do docker
```sh
	$ sudo update-rc.d docker defaults
```
Testar docker
```sh
	$ sudo service docker start
	$ sudo docker run hello-world
```

Comandos
-------------------------------
Listar imagens
```sh
	docker images

Lista container em funcionamento
```sh
	docker ps

Lista todos os container
```sh
	docker ps -a

Faz o download de imagens do dockerhub
```sh
	docker pull nome_imagem

Remover imagem
```sh
	docker rmi id_ou_nome_imagem

Acessar shell dentro do container (-i = interatividade, -t = tty terminal)
```sh
	docker exec -i -t blog-alura bash
	docker run -it ubuntu bash

Remover container (1por1)
```sh
	docker rm id_ou_nome_container

Remover container (vários) (-q = listar ids, -a = all), ($ = função shell script)
```sh
	docker rm $(docker ps -qa)

Parar container (se rodar o rm seguido do parametro -f não há a necessidade de parar o container para remove-lo)
```sh
	docker stop id_ou_nome_container

Verificar quanto cpu esta sendo utilizado por determinado container
```sh
	sudo docker stats id_ou_nome_container

Monitoramento de containers com cAdvisor (rodar comando linha-por-linha)
```sh
	sudo docker run \
	  --volume=/:/rootfs:ro \
  	  --volume=/var/run:/var/run:rw \
	  --volume=/sys:/sys:ro \
  	  --volume=/var/lib/docker/:/var/lib/docker:ro \
	  --publish=8080:8080 \
	  --detach=true \
	  --name=cadvisor \
	  google/cadvisor:latest

Obs.: Depois é só acessar localhost:8080

Criar container descartáveis (ubuntu = nome_imagem, --rm = destroi container após sair da execução do bash)
```sh
	docker run --rm -it ubuntu bash

Matar container (utilizado quando esta travado)
```sh
	docker kill id_ou_nome_container

Parar/retomar container
```sh
	docker stop id_ou_nome_container
	docker start id_ou_nome_container

Executar comando dentro do container (bash = comando)
```sh
	sudo docker exec -it nome_container_ativo bash

Executar comando dentro do container sem precisar acessa-lo (saida no bash local)
```sh
	sudo docker exec -it nome_container_ativo comando
```
	ex: sudo docker exec -it blog-alura echo $PATH
	ex: sudo docker exec -it blog-alura ps aux

Fazer commit para uma nova imagem
```sh
	docker commit -m "instalação do apache" [nome ou id do container] [imagem]/apache
```
	ex: docker commit -m "instalação do apache" id_ou_nome_container ubuntu/apache

Obs.: Neste exemplo, foi criado um container do ubuntu, aberto seu bash e instalado o apache, depois
foi realizado o commit deste container para criar a imagem ubuntu/apache

Iniciar apache
```sh
	apachectl start
```
Cria uma nova imagem, usando como base o arquivo Dockerfile (ubuntu/apache = nome da imagem, . = aqui
(caminho do Dockerfile))
```sh
	docker build -t ubuntu/apache .
	```
	
Subir container depois da build da imagem
```sh
	docker run -d -p 80:80 ubuntu/apache
```

Docker file (exemplo da imagem ubuntu instalando o apache e rodando em FOREGROUND na porta 80)
ex 1 (ubuntu + apache):
```sh
		FROM ubuntu
		RUN apt-get update && apt-get install -y apache2
		EXPOSE 80
		CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
ex 2 (mysql + apache):
```sh
		FROM mysql
		RUN apt-get update && apt-get install -y apache2
		EXPOSE 80
		CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Copiar arquivos para dentro do container (comando add)
ex 1 (ubuntu + apache):
```sh
		FROM ubuntu
		RUN apt-get update && apt-get install -y apache2
		ADD app/ /var/www/html
		EXPOSE 80
		CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

Copiar diretórios para dentro do container (comando copy)
ex 1 (ubuntu + apache):
```sh
		FROM ubuntu
		RUN apt-get update && apt-get install -y apache2
		ADD app/ /var/www/html
		COPY app/ /var/www/html/app
		EXPOSE 80
		CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```
Obs: Diferença entre copy e add
	A principal diferença entre o comando ADD e o comando COPY é que o
	ADD faz algumas coisas a mais como por exemplo aceitar arquivos comprimidos (.zip) e na
	hora de adicionar ele vai realizar a descompactação sozinho alem de permitir que o caminho
	do arquivo seja uma URL.

-- Criar os containers automaticamente utilizando docker-compose.yml
(db e blog = container, image = imagem base, environment = configuração de ambiente (ex:variável),
links = conexão com outro container, ports = mapeamento de portas)
ex1:
```sh
			db:
				image: mysql
				environment:
					- MYSQL_ROOT_PASSWORD=test123

				blog:
				image: wordpress
				environment:
					- WORDPRESS_DB_PASSWORD=test123
				links:
					- db:mysql
				ports:
					- 80:80
```
Obs: Será criado um container de nome "db". para ele será utilizada a imagem "mysql".
Além disso será feito o uso do environment passando a senha. O container "blog" tambémm
fará uso de uma imagem, de uma senha, de um link com o "db:mysql" e mapeadas as portas.
Salvamos e voltamos para o Terminal.

Atribuição de senha, usa-se sempre = e não : (dois pontos);

Executar arquivo docker-compose.yml (-d = rodar processos em background (sem bloquear shell))
```sh
		docker-compose up
		docker-compose up -d
```

Criar volumes no docker (volumes = diretório local, diretório mapeado no container)
Quando usamos o Docker e subimos um contêiner por padrão os dados não são mantidos,
criar um volume nada mais é do que mapear uma pasta local da nossa máquina para uma pasta
de um contêiner com a finalidade de manter os dados que foram criados no contêiner salvos.
Criar esse mapeamento é interessante quando por exemplo precisamos manter os dados do banco
de dados pois o mysql salva todas as informações em uma pasta dentro do contêiner que pode ser
mapeada para uma pasta da nossa máquina local.

ex1:
```sh
	db:
		image: mysql
		volumes:
			- ~/luucasor/blog_alura/database/:/var/lib/mysql/
		environment:
			- MYSQL_ROOT_PASSWORD=test123
	blog:
		image: wordpress
		environment:
			- WORDPRESS_DB_PASSWORD=test123
		links:
			- db:mysql
		ports:
			- 80:80
```sh

Matar e deletar containers com docker-compose
```sh
	docker-compose kill
	docker-compose rm
```
