
# Tutorial de Configuração do PM2 com NVM e WSL

Este guia detalha como instalar e configurar o PM2 em um ambiente WSL (Windows Subsystem for Linux) para gerenciar e monitorar várias aplicações Node.js. Com essa configuração, você evita a necessidade de abrir múltiplos terminais para cada aplicação.

## Passo 1: Instalando o WSL

Primeiro, vamos instalar o WSL para rodar o PM2 no ambiente Linux. Execute o seguinte comando no CMD com permissões de administrador:

```bash
wsl --install -d Ubuntu
```

1. Após a instalação, reinicie o computador.
2. Quando o sistema voltar, será solicitado que você configure um usuário e uma senha para o Ubuntu no WSL.
3. Caso o Ubuntu não abra automaticamente, procure por "Ubuntu" na barra de tarefas e inicie o terminal.

A partir deste ponto, todos os comandos deverão ser executados no terminal do Ubuntu. Se tiver problemas ao colar comandos, clique com o botão direito no terminal para inserir o texto.

### Atualizando o Ubuntu

Após a instalação do WSL, atualize o sistema com o seguinte comando:

```bash
sudo apt update && sudo apt upgrade
```

Caso de algum erro faça o seguinte passo a passo:

Va até esse arquivo:
```bash
sudo nano /etc/resolv.conf
```

Adicione isto no fim do arquivo:

```text
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```
Salve e feche o arquivo (pressione CTRL + X, depois Y e Enter).

E reinicie o wsl usando o seguinte comando no CMD do seu windows:

```bash
wsl --shutdown
```

E agora abra o wsl novamente e estará arrumado o problema.

## Passo 2: Instalando o NVM (Node Version Manager)

Para gerenciar diferentes versões do Node.js, vamos instalar o NVM:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

1. Após a instalação do NVM, feche e reabra o terminal do Ubuntu.

**Recomendação**: Para este tutorial, sugiro a versão 20.18 do Node.js. Para instalá-la, execute:

```bash
nvm install 20.18
```

Para ver todas as versões de Node.js instaladas:

```bash
nvm ls
```

Para mudar para uma versão específica do Node.js:

```bash
nvm use 20.18
```

Verifique se o Node.js e o npm foram instalados corretamente com:

```bash
node -v
npm -v
```

**Nota**: Se ocorrer algum erro, repita a instalação do Node.js com o NVM.

## Passo 3: Instalando o PM2

Agora, instale o PM2 globalmente:

```bash
npm i -g pm2
```

Caso veja alguma mensagem de erro, tente instalar outra versão do Node.js e repita o processo. Para testar a instalação do PM2, use:

```bash
pm2 ping
```

Se houver resposta, a instalação foi bem-sucedida.

## Passo 4: Iniciando Projetos com o PM2

Navegue até o diretório da sua aplicação e compile-a com o comando configurado, por exemplo:

```bash
npm run build
```

Depois, inicie a aplicação com o PM2 usando o comando abaixo. Ele abstrai o comando `npm run start` para ser gerenciado pelo PM2:

```bash
pm2 start npm --name "minha-app" -- start
```

Sua aplicação agora está sendo gerenciada pelo PM2!

## Passo 5: Espelhando a porta do WSL

Agora vamos liberar e espelhar a porta do seu WSL. Primeiro use o seguinte comando:

```bash
sudo apt install ufw
```

Após isso vamos liberar a porta `8096` como exemplo:

```bash
sudo ufw allow 8096/tcp
```

Não esqueca de liberar todas as portas que você for utilizar.

Agora no Windows crie um arquivo .bat e cole o seguinte conteudo dentro dele:

```bash
@echo off
for /f "tokens=*" %%i in ('wsl -d Ubuntu hostname -I') do set ip_wsl=%%i

REM Remove redirecionamentos existentes para as portas especificadas
netsh interface portproxy delete v4tov4 listenport=8096

REM Adiciona novos redirecionamentos para as portas especificadas
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=8096 connectaddress=%ip_wsl% connectport=8096

echo Redirecionamento de portas configurado com sucesso para IP %ip_wsl%
pause

```

No exemplo acima estou espelhando a porta 8096 do meu computador com a porta 8096 do WSL. E como não é possivel deixar o ip do WSL fixo, esse .bat serve para toda vez que o ip mudar vc apenas executar ele e pronto. Eu recomendo que faça o seu computador executar esse .bat toda vez que inicilizar o sistema.

## Comandos Úteis do PM2

### Monitoramento em Tempo Real

Para monitorar o uso de CPU e memória dos processos gerenciados pelo PM2:

```bash
pm2 monit
```

### Listagem de Processos

Para ver o status atual de todas as aplicações, junto com o consumo de CPU e memória:

```bash
pm2 ls
```

### Comandos Adicionais

Aqui estão alguns comandos adicionais do PM2 que podem ajudar no gerenciamento das aplicações:

- **Reiniciar um processo**: Reinicia uma aplicação específica pelo nome ou ID.
  ```bash
  pm2 restart <nome-ou-id>
  ```

- **Parar um processo**: Para uma aplicação específica pelo nome ou ID.
  ```bash
  pm2 stop <nome-ou-id>
  ```

- **Remover um processo**: Exclui um processo da lista do PM2.
  ```bash
  pm2 delete <nome-ou-id>
  ```

- **Salvar processos**: Salva a lista de processos atuais para serem reiniciados automaticamente após uma reinicialização do sistema.
  ```bash
  pm2 save
  ```

- **Carregar processos salvos**: Carrega e inicia automaticamente os processos salvos anteriormente.
  ```bash
  pm2 resurrect
  ```

- **Logs das Aplicações**: Exibe logs em tempo real de todas as aplicações gerenciadas pelo PM2.
  ```bash
  pm2 logs
  ```

- **Reiniciar todos os processos**: Reinicia todas as aplicações em execução.
  ```bash
  pm2 restart all
  ```

## Links Úteis

- [Documentação do NVM](https://github.com/nvm-sh/nvm)
- [Documentação do PM2](https://pm2.keymetrics.io/docs/usage/quick-start/)
- [Vídeo recomendado sobre PM2](https://www.youtube.com/watch?v=zi8qHEL-Ilk)
