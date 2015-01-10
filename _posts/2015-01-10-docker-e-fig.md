---
layout: post
title: Configuração do Docker e Fig
---

[![Na minha máquina funciona][minha-maquina-img]][minha-maquina-link]

Uma das formas para melhorar a qualidade e diminuir o tempo de entrega de novos recursos é através da padronização de 
ambientes de desenvolvimento, homologação e produção. É a velha história do "funciona na minha máquina.
É aí que entra o [Docker]

Com o Docker conseguimos construir um ambiente isolado através do conceito de *containers*. Um *container* é um processo 
virtualizado, onde ao invés de configurar uma máquina virtual instalando todos os serviços que precisamos, configuramos
uma máquina virtual para cada serviço. Isso só é possível pois o docker utiliza o [Linux Containers], onde o *container*
compartilha parte do kernel da máquina hospedeira de uma forma mais inteligente, não possuindo assim o overhead de emular
todo o hardware.

Assim resolvemos o problema de manter um ambiente confiável em qualquer ambiente, seja Linux ou Windows, mas como cada
serviço possui um container a parte, surge a dificuldade da orquestração destes serviços. Precisamos inicializar todos
os serviços de forma independente, além de definir os volumes compartilhados entre a máquina hospedeira e o container,
conectar os containers entre si, através da especificação do IP e porta correspondentes, etc. É nesse ponto que o [Fig]
entra em ação.

## Configuração do Fig

O Fig extende o Docker, permitindo que todos os containers necessários sejam iniciados, encerrados, excluídos, etc. com 
apenas um comando. A configuração é tão simples quanto o Docker. Precisamos de um arquivo **fig.yml** na raiz do projeto,
como o a seguir:

{% highlight yaml %}
web:
  build: .
  command: php -S 0.0.0.0:8000 -t /code
  ports:
    - "80:8000"
  links:
    - db
  volumes:
    - .:/code
db:
  image: orchardup/mysql
  environment:
    MYSQL_DATABASE: dbname
{% endhighlight %}

### Container Web

<dl>
  <dt>web:</dt>
  <dd>Identificação do container. O nome <strong>web</strong> é arbitrário.</dd>
  
  <dt>build: .</dt>
  <dd>Neste caso, temos um <strong>Dockerfile</strong> definido no mesmo diretório do fig.yml, então este container será construído a partir do diretório atual.</dd>
  
  <dt>command: php -S 0.0.0.0:8000 -t /code</dt>
  <dd>Define o <strong>CWD</strong> do container. Neste caso utilizaremos o
    <a href="http://php.net/manual/pt_BR/features.commandline.webserver.php">servidor embarcado do PHP</a> na porta 8000
    apontando para o diretório <strong>/code</strong></dd>
    
  <dt>ports: -"8000:8000"</dt>
  <dd>Vinculamos a porta 8000 do container à porta 80 da máquina hospedeira. Utilizaremos http://localhost/ para acessar
    o servidor do container posteriormente.</dd>
  
  <dt>links: - db</dt>
  <dd>Essa é a parte realmente interessante do Fig. Aqui vinculamos o container <strong>web</strong> com o container
    <strong>db</strong>, que será definido posteriormente.</dd>
    
  <dt>volumes: - .:/code</dt>
  <dd>Definimos que a pasta <strong>/code</strong> do container, apontará para o diretório atual da máquina hospedeira.</dd>
</dl>


### Container DB

<dl>
  <dt>db:</dt>
  <dd>Este container será chamado <strong>db</strong></dd>
  
  <dt>image: orchardup/mysql</dt>
  <dd>Neste caso, queremos utilizar uma imagem encontrada no <a href="https://registry.hub.docker.com/">Docker Hub Repository</a></dd>
  
  <dt>environment: MYSQL_DATABASE: dbname</dt>
  <dd>A imagem <a href="https://registry.hub.docker.com/u/orchardup/mysql/">orchardup/mysql</a> utiliza variáveis de
    ambiente para se configurar, neste caso definimos o nome do banco de dados que utilizaremos.</dd>
</dl>

## Os nomes dos containers, serão seus hosts

No **fig.yml** definimos dois containers, **web** e **db**. Provavelmente um arquivo de configuração da aplicação hospedada
no container **web** precisará se comunicar com o banco de dados em algum momento. Os dados de conexão são definidos pelas
variáveis de ambiente do container, conforme visto acima, porém o IP do servidor é atribuído pelo próprio Docker. 

Utilizando uma definição do próprio Docker, o fig permite conectar dois containers através da configuração **links**.
Ao definir esta conexão, o Docker adiciona ao arquivo **/etc/hosts** do container, o IP e o nome do container definido
no fig, como a seguir:

{% highlight text %}
172.17.0.7 web
172.17.0.8 db
{% endhighlight %}

Dessa forma, nosso arquivo de configuração ficaria semelhante ao seguinte:

{% highlight php %}
<?php

$db = array(
  'dbname' => 'dbname',
  'dbuser' => 'root',
  'dbpass' => 'mysecretpassword',
  'dbhost' => 'db'
);
{% endhighlight %}

A linha importante neste arquivo é `'dbhost' => 'db'`, onde apontamos como host, a string 'db'.

## Iniciando sua aplicação

O Fig funciona de forma muito semelhante ao Docker. Após configurar o fig.yml, é só iniciar tudo:

{% highlight bash %}
$ fig up
{% endhighlight %}


Você pode passar a opção `fig up -d` para rodar sua aplicação em background. Outros comandos do Docker
também se aplicam:

* `fig stop`: Para os containers que estão rodando.
* `fig rm`: Remove os containers parados.
* `fig restart`: Reinicia os containers.
* `fig ps`: Lista os containers da sua aplicação.

Os comandos do fig evitam que você precise executar diversas vezes o mesmo comando docker, para cada um de seus
containers ou criar shell scripts para rodar sua aplicação.


[minha-maquina-img]: http://vidadeprogramador.com.br/wp-content/uploads/2011/06/tirinha124.png
[minha-maquina-link]: http://vidadeprogramador.com.br/2011/06/03/nao-funciona-no-servidor/
[SocialBase]: http://www.socialbase.com.br/
[Vagrant]: https://www.vagrantup.com/
[Docker]: http://www.docker.com/
[Linux Containers]: https://linuxcontainers.org/
[Fig]: http://www.fig.sh/
[servidor embarcado do PHP]: http://php.net/manual/pt_BR/features.commandline.webserver.php

