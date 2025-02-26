#!/bin/bash

# Função para exibir mensagens coloridas
log() {
    echo -e "\e[32m[INFO]\e[0m $1"
}

die() {
    echo -e "\e[31m[ERROR]\e[0m $1" >&2
    exit 1
}

if [ -z "$1" ]; then
    die "Uso: $0 [master|worker]"
fi

NODE_TYPE=$1

log "Verificando se o Docker está instalado..."
if ! command -v docker &>/dev/null; then
    die "Docker não está instalado. Instale o Docker e tente novamente."
fi

log "Reiniciando o Docker..."
sudo systemctl restart docker

log "Liberando portas do Docker Swarm..."
sudo ufw allow 2377/tcp
sudo ufw allow 7946/tcp
sudo ufw allow 7946/udp
sudo ufw allow 4789/udp
sudo ufw reload

if [ "$NODE_TYPE" == "master" ]; then
    log "Verificando se o Swarm está inicializado..."
    if ! docker info | grep -q "Swarm: active"; then
        log "Inicializando o Docker Swarm..."
        docker swarm init --advertise-addr $(hostname -I | awk '{print $1}') || die "Falha ao inicializar o Swarm."
    fi
    log "Swarm já está ativo. Exibindo token de join para workers:"
    docker swarm join-token worker | grep "docker swarm join"
elif [ "$NODE_TYPE" == "worker" ]; then
    log "Verificando conexão com o master..."
    read -p "Digite o IP do master: " MASTER_IP
    if ! nc -zv $MASTER_IP 2377; then
        die "Falha na conexão com o master. Verifique a rede e tente novamente."
    fi
    log "Obtendo token do master..."
    TOKEN=$(ssh user@$MASTER_IP "docker swarm join-token worker -q")
    log "Ingressando no cluster..."
    docker swarm join --token $TOKEN $MASTER_IP:2377 || die "Falha ao ingressar no Swarm."
else
    die "Tipo de nó inválido. Use 'master' ou 'worker'."
fi

log "Configuração concluída com sucesso!"
