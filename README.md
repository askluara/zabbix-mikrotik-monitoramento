# 📡 Monitoramento de Ativos de Rede com Zabbix Server e MikroTik CHR

Projeto de monitoramento de infraestrutura de rede utilizando o **Zabbix Appliance** e o **MikroTik CHR (Cloud Hosted Router)**. A solução utiliza o protocolo **SNMP v2c** para coletar métricas do roteador virtualizado e o serviço **SMTP/Gmail** para o envio automatizado de alertas por e-mail em HTML.

O objetivo é demonstrar a configuração, a integração e a validação de uma solução de monitoramento capaz de identificar eventos na infraestrutura de rede e notificar os responsáveis por meio de alertas automatizados.

---

## 📑 Sumário

- [1. Visão geral do projeto](#-1-visão-geral-do-projeto)
- [2. Arquitetura da infraestrutura](#-2-arquitetura-da-infraestrutura)
- [3. Tecnologias utilizadas](#-3-tecnologias-utilizadas)
- [4. Pré-requisitos](#-4-pré-requisitos)
- [5. Configuração do MikroTik CHR](#-5-configuração-do-mikrotik-chr)
- [6. Configuração do SNMP](#-6-configuração-do-snmp)
- [7. Integração com o Zabbix](#-7-integração-com-o-zabbix)
- [8. Configuração de alertas por e-mail](#-8-configuração-de-alertas-por-e-mail)
- [9. Validação do monitoramento](#-9-validação-do-monitoramento)
- [10. Solução de problemas](#-10-solução-de-problemas)
- [11. Segurança e boas práticas](#-11-segurança-e-boas-práticas)
- [12. Estrutura do repositório](#-12-estrutura-do-repositório)

---

## 🎯 1. Visão geral do projeto

O projeto implementa uma solução de monitoramento para um roteador MikroTik CHR, utilizando o Zabbix como plataforma central de coleta, visualização e notificação de eventos.

### Funcionalidades

- Monitoramento de disponibilidade do roteador.
- Coleta de métricas por meio do protocolo SNMP v2c.
- Acompanhamento do uso de CPU e memória.
- Monitoramento do tráfego das interfaces de rede.
- Verificação do tempo de atividade (*uptime*).
- Identificação de situações anormais por meio de triggers.
- Envio de alertas automatizados via SMTP/Gmail.
- Notificações por e-mail com conteúdo HTML personalizado.

### Fluxo de funcionamento

1. O MikroTik CHR disponibiliza informações de monitoramento por SNMP.
2. O Zabbix Server consulta o roteador por meio da rede configurada.
3. Os dados coletados são armazenados e apresentados na interface web do Zabbix.
4. As triggers avaliam as condições configuradas para identificar problemas.
5. Quando um evento atende às condições de notificação, o Zabbix pode enviar um alerta por e-mail.

---

## 🏗️ 2. Arquitetura da infraestrutura

A infraestrutura utiliza máquinas virtuais para isolar os serviços e permitir a comunicação entre o servidor de monitoramento e o roteador virtualizado.

### Topologia lógica

```text
                    INTERNET
                       |
                 Adaptador NAT
                       |
              +------------------+
              |   MikroTik CHR   |
              |    RouterOS      |
              |    SNMP v2c      |
              +------------------+
                       |
                Rede de monitoramento
                 Host-Only / LAN
                       |
              +------------------+
              |   Zabbix Server  |
              | Coleta de métricas|
              | Triggers e eventos|
              +------------------+
                       |
                  SMTP / Gmail
                       |
                 +-----------+
                 |  E-mail   |
                 |  de alerta|
                 +-----------+
```

> **Observação:** o diagrama representa a arquitetura lógica. A comunicação entre as máquinas depende do endereçamento IP, das interfaces configuradas e das regras de firewall do ambiente.

### 📋 Especificações da VM — MikroTik CHR

| Componente | Especificação |
|---|---|
| Sistema operacional | MikroTik CHR / RouterOS |
| Versão | RouterOS v6.x ou v7.x, conforme a imagem utilizada |
| Arquitetura | x86_64 |
| Virtualização | VirtualBox, VMware Workstation ou Proxmox |
| Processador | 1 vCPU |
| Memória RAM | 128 MB, sujeito à compatibilidade com a versão |
| Armazenamento | Imagem virtual VDI ou VMDK, conforme o hipervisor |
| Interface WAN | NAT para acesso à internet, quando necessário |
| Interface de monitoramento | Host-Only ou rede LAN dedicada |

**Importante:** os 128 MB de RAM e 128 MB de armazenamento são valores de referência do ambiente proposto, não requisitos universais para todas as versões do RouterOS. Verifique as recomendações da versão utilizada.

### Endereçamento IP de exemplo

| Equipamento | Endereço IP de exemplo |
|---|---|
| MikroTik CHR | `192.168.88.1/24` |
| Zabbix Server | `192.168.88.10/24` |
| Rede de monitoramento | `192.168.88.0/24` |

Os endereços devem ser adaptados à rede efetivamente configurada no ambiente.

---

## 🧰 3. Tecnologias utilizadas

- **MikroTik CHR:** roteador virtualizado responsável por disponibilizar as métricas de rede.
- **RouterOS:** sistema operacional do MikroTik.
- **Zabbix Appliance / Server:** plataforma de monitoramento, coleta de dados e gerenciamento de eventos.
- **SNMP v2c:** protocolo utilizado para consultar informações do dispositivo.
- **SMTP/Gmail:** serviço utilizado para o envio de notificações.
- **HTML:** formato utilizado na personalização dos e-mails de alerta.
- **VirtualBox / VMware / Proxmox:** plataformas de virtualização compatíveis com o ambiente proposto.

---

## ✅ 4. Pré-requisitos

Antes de iniciar a implantação, verifique se os seguintes itens estão disponíveis:

- [ ] Hipervisor instalado e funcionando.
- [ ] Imagem oficial do MikroTik CHR.
- [ ] Zabbix Appliance ou Zabbix Server acessível pela interface web.
- [ ] Interfaces de rede configuradas para permitir a comunicação entre as VMs.
- [ ] Endereço IP configurado no MikroTik e no Zabbix Server.
- [ ] Porta UDP 161 liberada para consultas SNMP entre o Zabbix Server e o roteador.
- [ ] Template de monitoramento compatível com a versão do Zabbix.
- [ ] Conta Google com as condições necessárias para autenticação SMTP.
- [ ] Senha de app do Google, caso exigida para o método de autenticação utilizado.

---

## ⚙️ 5. Configuração do MikroTik CHR

### 5.1. Obter a imagem

Baixe a imagem do MikroTik CHR diretamente do site oficial:

- [Download do MikroTik CHR](https://mikrotik.com/download)

Escolha o formato compatível com o hipervisor utilizado, como VDI ou VMDK.

### 5.2. Criar a máquina virtual

Configure a VM de acordo com as especificações do ambiente.

1. Crie uma nova máquina virtual no hipervisor.
2. Selecione a imagem do MikroTik CHR como disco de inicialização.
3. Configure a memória RAM e a quantidade de CPUs.
4. Adicione os adaptadores de rede necessários.
5. Inicie a máquina virtual e acesse o console do RouterOS.

O tipo de sistema operacional exibido pelo hipervisor pode variar. Utilize a configuração compatível com a imagem CHR baixada.

### 5.3. Configurar o endereço IP

No terminal do RouterOS, identifique as interfaces disponíveis:

```routeros
/interface print
```

Configure o endereço IP na interface conectada à rede de monitoramento. Neste exemplo, considera-se que essa interface é `ether1`:

```routeros
/ip address add address=192.168.88.1/24 interface=ether1
```

**Atenção:** confirme qual interface está conectada à rede de monitoramento antes de executar o comando. Se `ether1` estiver conectada à WAN, utilize a interface correta para evitar conflitos de endereçamento.

Verifique o endereço configurado:

```routeros
/ip address print
```

Teste a conectividade com o servidor de monitoramento:

```routeros
/ping 192.168.88.10
```

Substitua o endereço pelo IP real do Zabbix Server.

---

## 📡 6. Configuração do SNMP

O SNMP permite que o Zabbix consulte informações do roteador sem instalar agentes adicionais no MikroTik.

### 6.1. Configurar a comunidade SNMP

No terminal do RouterOS, configure uma comunidade de leitura dedicada ao monitoramento.

Exemplo para RouterOS v6/v7, sujeito às diferenças de sintaxe e disponibilidade da versão:

```routeros
/snmp community add name=monitor-lab addresses=192.168.88.10/32 read-access=yes
/snmp set enabled=yes
```

Nesse exemplo:

- `monitor-lab`: nome da comunidade SNMP.
- `192.168.88.10/32`: endereço autorizado do Zabbix Server.
- `read-access=yes`: permite consultas de leitura.
- `enabled=yes`: habilita o serviço SNMP.

Se já existir uma comunidade com esse nome, consulte as comunidades cadastradas antes de criar outra.

```routeros
/snmp community print
/snmp print
```

> **Nota de compatibilidade:** a propriedade `read-access` e os parâmetros disponíveis devem ser confirmados na versão específica do RouterOS. Ajuste a sintaxe caso necessário.

### 6.2. Porta e protocolo

| Parâmetro | Valor |
|---|---|
| Protocolo | SNMP v2c |
| Porta | UDP 161 |
| Comunidade | `monitor-lab` |
| Origem autorizada | IP do Zabbix Server |
| Permissão | Leitura |

### 6.3. Validar o SNMP

A partir do servidor de monitoramento, execute um teste utilizando uma ferramenta SNMP, caso esteja instalada:

```bash
snmpwalk -v2c -c monitor-lab 192.168.88.1 1.3.6.1.2.1.1
```

O comando consulta informações básicas do sistema, como descrição, identificação e tempo de atividade.

Substitua `192.168.88.1` pelo endereço real do MikroTik.

Se o comando retornar informações do dispositivo, isso indica que a consulta SNMP está funcionando. Se não houver resposta, verifique a conectividade IP, a comunidade, a versão do protocolo e as regras de firewall.

---

## 📊 7. Integração com o Zabbix

### 7.1. Importar o template

1. Acesse a interface web do Zabbix.
2. Entre na seção **Data collection → Templates**. Em versões anteriores, o caminho pode aparecer como **Configuration → Templates**.
3. Clique em **Import**.
4. Selecione o arquivo `template_mikrotik.json`, caso esteja disponível no repositório.
5. Revise as opções de importação.
6. Confirme a importação.

**Atenção:** o arquivo precisa existir no repositório e utilizar um formato compatível com a versão instalada do Zabbix. Caso utilize um template oficial, confirme se ele é apropriado para o RouterOS e para SNMP v2c.

### 7.2. Cadastrar o host

1. Acesse **Data collection → Hosts**.
2. Clique em **Create host**.
3. Preencha os dados do equipamento.

| Campo | Valor de exemplo |
|---|---|
| Host name | `MikroTik-CHR` |
| Host group | `Routers` |
| Template | Template MikroTik compatível com SNMP |
| Interface | SNMP |
| IP address | `192.168.88.1` |
| Port | `161` |
| SNMP version | `SNMPv2` |
| SNMP community | `monitor-lab` |

4. Associe o template ao host.
5. Salve o cadastro.

O nome do grupo e do template pode variar conforme os objetos existentes no servidor.

### 7.3. Verificar a coleta

Após cadastrar o host:

1. Acesse a lista de hosts.
2. Confirme se a interface SNMP está configurada corretamente.
3. Verifique se os itens de monitoramento estão coletando dados.
4. Consulte **Monitoring → Latest data** para visualizar os valores recebidos.
5. Verifique a disponibilidade do host e possíveis erros de coleta.

As métricas disponíveis dependem do template importado. Entre as informações esperadas estão CPU, memória, tráfego das interfaces e uptime, desde que sejam suportadas pelo template e pela versão do RouterOS.

---

## 📧 8. Configuração de alertas por e-mail

O Zabbix pode utilizar o Gmail como servidor SMTP para encaminhar notificações geradas a partir de eventos e ações configuradas.

### 8.1. Preparar a conta Google

Para contas Google com autenticação por senha de app, gere a credencial nas configurações de segurança da conta, seguindo os requisitos aplicáveis à conta institucional.

A senha de app é uma credencial sensível e não deve ser publicada no repositório.

### 8.2. Configurar o Media Type

No Zabbix, acesse **Alerts → Media types** ou a seção equivalente na versão instalada.

Configure o método de envio de e-mail:

| Parâmetro | Valor |
|---|---|
| Media type | Email |
| SMTP server | `smtp.gmail.com` |
| SMTP server port | `465` |
| Connection security | SSL/TLS |
| Authentication | Username and password |
| Username | Endereço de e-mail autorizado |
| Password | Senha de app, quando aplicável |
| Status | Enabled |

Utilize o endereço institucional autorizado para o envio, em vez de expor credenciais no código ou na documentação pública.

**Importante:** a porta 465 utiliza conexão SMTP com TLS implícito. Se optar pela porta 587, a configuração normalmente utiliza STARTTLS. Não misture os métodos de segurança.

Salve as alterações e utilize o recurso **Test**, se disponível, para validar a configuração SMTP.

### 8.3. Configurar o destinatário

Para que o alerta seja entregue, também é necessário:

1. Cadastrar o endereço de e-mail do destinatário no perfil do usuário do Zabbix.
2. Associar a mídia Email ao usuário.
3. Definir o período de atividade e as severidades desejadas.
4. Criar ou revisar as ações de notificação para os eventos.
5. Confirmar que o usuário possui as permissões e configurações necessárias para receber os alertas.

A configuração do servidor SMTP, isoladamente, não garante o envio automático de notificações: as mídias, os usuários e as ações também precisam estar configurados.

### 8.4. Personalizar o conteúdo em HTML

O corpo da mensagem pode utilizar um template HTML personalizado para apresentar as informações do evento de maneira organizada.

Exemplo de conteúdo:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Alerta de Monitoramento</title>
</head>
<body style="font-family: Arial, sans-serif; color: #333333;">
    <h2>Alerta de Monitoramento de Rede</h2>

    <p><strong>Ativo:</strong> {HOST.NAME}</p>
    <p><strong>IP:</strong> {HOST.IP}</p>
    <p><strong>Severidade:</strong> {EVENT.SEVERITY}</p>
    <p><strong>Evento:</strong> {EVENT.NAME}</p>
    <p><strong>Data e hora:</strong> {EVENT.DATE} {EVENT.TIME}</p>

    <p>Verifique o painel do Zabbix para obter mais informações
    sobre o evento identificado.</p>
</body>
</html>
```

**Observações sobre as macros:**

- As macros disponíveis dependem do contexto da mensagem e da versão do Zabbix.
- `{ITEM.LASTVALUE}` só deve ser utilizado quando o contexto da notificação permitir a resolução do item correspondente.
- `{HOST.IP}` pode depender do contexto do host e da interface selecionada.
- Para notificações de recuperação, utilize as macros adequadas ao evento e ao tipo de mensagem.
- O template HTML deve ser configurado no corpo da mensagem do Media Type ou no template de mensagem correspondente, conforme a versão do Zabbix.

---

## 🧪 9. Validação do monitoramento

Esta etapa verifica se o roteador está sendo monitorado e se os alertas são enviados quando ocorre um evento.

### 9.1. Validar a conectividade

Confirme se o Zabbix Server alcança o MikroTik pela rede de monitoramento.

Verifique:

- Endereços IP e máscaras de rede.
- Interface correta no MikroTik.
- Conectividade entre as VMs.
- Regras de firewall.
- Acesso à porta UDP 161.

### 9.2. Validar a coleta SNMP

Execute o teste com `snmpwalk` e verifique se o servidor recebe respostas do roteador.

Em seguida, consulte os dados mais recentes do host na interface web do Zabbix.

### 9.3. Validar as triggers

1. Acesse **Data collection → Hosts** ou a seção equivalente na versão instalada.
2. Abra as triggers associadas ao host ou ao template.
3. Identifique a trigger de CPU, caso ela exista.
4. Verifique a expressão e o limite configurado.
5. Confirme se o item utilizado pela trigger está recebendo dados.

**Não é recomendado alterar diretamente uma trigger herdada de um template em produção.** Para realizar o teste, prefira criar uma trigger temporária de laboratório ou uma cópia independente.

Se utilizar um limite de CPU extremamente baixo, como `1%`, o evento poderá permanecer em estado de problema continuamente. Além disso, a coleta e a avaliação das triggers dependem dos intervalos configurados, portanto não há garantia de que o evento aparecerá exatamente após um minuto.

### 9.4. Validar o recebimento do alerta

Após a geração de um evento que satisfaça as condições da ação:

1. Confirme se o evento aparece no painel de monitoramento.
2. Verifique se a ação de notificação foi executada.
3. Consulte o histórico de ações e mensagens do Zabbix.
4. Confira a caixa de entrada e a pasta de spam do destinatário.
5. Verifique o conteúdo HTML e as informações apresentadas.
6. Restaure ou remova a trigger temporária após a validação.

### 9.5. Checklist final

- [ ] O MikroTik CHR está ligado e acessível.
- [ ] A interface de rede possui o endereço IP esperado.
- [ ] O SNMP está habilitado.
- [ ] O teste SNMP retorna informações do roteador.
- [ ] O host está cadastrado no Zabbix.
- [ ] O template está associado e coletando métricas.
- [ ] As triggers estão avaliando os dados.
- [ ] O Media Type Email está habilitado.
- [ ] O teste SMTP foi concluído com sucesso.
- [ ] O destinatário está configurado.
- [ ] O alerta de teste foi recebido corretamente.
- [ ] As configurações temporárias de teste foram removidas.

---

## 🛠️ 10. Solução de problemas

| Problema | Possível causa | Ação recomendada |
|---|---|---|
| MikroTik inacessível | IP incorreto ou rede incompatível | Conferir endereçamento, interfaces e conectividade |
| SNMP sem resposta | Serviço desabilitado ou comunidade incorreta | Conferir `/snmp print` e `/snmp community print` |
| Timeout no SNMP | Firewall bloqueando UDP 161 | Verificar regras de firewall e restrições da comunidade |
| Host indisponível no Zabbix | Interface SNMP ou credenciais incorretas | Revisar IP, porta, versão e comunidade |
| Métricas não aparecem | Template incompatível ou itens sem coleta | Verificar itens, macros, template e logs |
| E-mail não é enviado | SMTP incorreto ou credenciais inválidas | Conferir porta, segurança, autenticação e senha de app |
| E-mail não chega ao destinatário | Mídia ou ação não configurada | Revisar usuário, endereço, período ativo e severidades |
| Alerta não é disparado | Trigger não atingida ou ausência de dados | Revisar a expressão e o intervalo de coleta |
| Macros aparecem sem valores | Macro incompatível com o contexto | Conferir as macros disponíveis para a mensagem |

---

## 🔐 11. Segurança e boas práticas

Para manter o ambiente seguro e facilitar sua reprodução, recomenda-se:

- Não utilizar a comunidade padrão `public` em ambientes expostos.
- Restringir o acesso SNMP ao endereço IP do servidor de monitoramento.
- Permitir somente consultas de leitura quando esse for o objetivo.
- Não expor a porta UDP 161 diretamente à internet.
- Não publicar senhas, senhas de app ou outros segredos no Git.
- Utilizar variáveis de ambiente ou mecanismos seguros de armazenamento de credenciais quando aplicável.
- Remover credenciais de arquivos de configuração antes de compartilhar o repositório.
- Utilizar triggers de teste independentes para evitar alterações acidentais em configurações de produção.
- Avaliar o uso de SNMPv3 quando forem necessários mecanismos adicionais de autenticação e privacidade.

> **Aviso:** SNMP v2c utiliza uma comunidade enviada sem criptografia. Portanto, a restrição de acesso à rede é fundamental, especialmente em ambientes que transportam informações sensíveis.

---

## 📁 12. Estrutura do repositório

Uma estrutura sugerida para organizar os artefatos do projeto é:

```text
monitoramento-ativos-rede/
├── README.md
├── templates/
│   └── template_mikrotik.json
├── config/
│   └── snmp.example.txt
├── docs/
│   └── arquitetura.md
└── .gitignore
```

**Descrição dos diretórios:**

- `README.md`: documentação principal e instruções de implantação.
- `templates/`: templates do Zabbix exportados para importação.
- `config/`: exemplos de configuração sem credenciais reais.
- `docs/`: documentação complementar da arquitetura e dos testes.
- `.gitignore`: regras para impedir que arquivos locais ou informações sensíveis sejam enviados ao repositório.

Os arquivos apresentados são uma estrutura sugerida. Inclua somente os arquivos que realmente existirem no projeto e que sejam necessários para sua reprodução.

---

## 📚 Referências oficiais

- [MikroTik — CHR](https://mikrotik.com/download)
- [MikroTik — Documentação do RouterOS](https://help.mikrotik.com/docs/)
- [MikroTik — SNMP](https://help.mikrotik.com/docs/spaces/ROS/pages/8978519/SNMP)
- [Zabbix — Documentação oficial](https://www.zabbix.com/documentation/current/en/manual)
- [Google — Senhas de app](https://support.google.com/accounts/answer/185833?hl=pt-BR)

---

## 👩‍💻 Considerações finais

Este projeto demonstra a integração entre roteamento virtualizado, monitoramento de infraestrutura, coleta de métricas via SNMP e envio automatizado de notificações.

A solução permite acompanhar o comportamento do MikroTik CHR, identificar eventos relevantes e centralizar informações operacionais no Zabbix, contribuindo para a compreensão de práticas de monitoramento, administração de redes e observabilidade de infraestrutura.
