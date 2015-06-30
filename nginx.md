# Instalando Nginx *atualizado* no CentOS/Amazon Linux AMI

Para instalar o **Nginx (engine X)** no CentOS/Amazon Linux AMI basta executar o comando `yum install nginx`, porém, a versão que está nos repositórios da Amazon está desatualizada. Se você precisar de uma versão mais atual, ou até mesmo a mais recente, você precisa instalar e compilar manualmente, o processo é um tanto quanto simples. Let's go?

Conecte-se via `ssh` na instância que você quer instalar o **Nginx**, feito isso, troque o usuário para `root`.

```bash
$ sudo su
$ cd
```

Atualize o sistema.

```bash
$ yum update
```

Instale as dependências necessárias para instalar e compilar o **Nginx**.

```bash
$ yum install pcre-devel zlib-devel openssl-devel
$ yum install gcc
$ yum install make
```

Baixe a versão do Nginx desejada (no exemplo abaixo, a última versão até a data de publicação dessa wiki era a *1.9.2*).

```bash
$ wget http://nginx.org/download/nginx-1.9.2.tar.gz
```

Descompacte o arquivo.

```bash
$ tar xzf nginx-1.9.2.tar.gz
```

Navegue até o diretório onde o **Nginx** foi descompactado.

```bash
$ cd nginx-1.9.2
```

Execute o seguinte comando para definir o diretório de instalação:

```bash
$ ./configure --sbin-path=/usr/sbin --with-http_ssl_module
```

Execute os comandos abaixo para compilar e instalar o **Nginx**

```bash
$ make
$ make install
```

Feito, **Nginx** instalado, agora é só configurar!

## Configurando o Nginx

Crie o arquivo `nginx` no diretório `init.d` com o comando abaixo:

```bash
$ touch /etc/init.d/nginx
```

**Copie o conteúdo abaixo para o arquivo criado anteriormente.**

```shell
#!/bin/sh
#
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:     /usr/local/nginx/logs/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/sbin/nginx"
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
 
lockfile=/var/lock/subsys/nginx
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

Execute o comando abaixo para tornar o arquivo `nginx` executável.

```bash
$ chmod 755 /etc/init.d/nginx
```

Pronto, agora é só testar para ver se está tudo ok.

```bash
$ /etc/init.d/nginx stop
$ /etc/init.d/nginx start
$ /etc/init.d/nginx restart
```

Reiniciando o terminal/cmd (você sabe, desconectar do ssh e conectar novamente) será possível executar o **Nginx** e suas ações com os comandos abaixo.

```bash
$ service nginx start
$ service nginx stop
$ service nginx restart
```

:octocat: 
