# Emissão de Certificado de Máquina fora do Domínio (AD CS)

## Visão Geral

Este documento descreve o procedimento para emissão manual de certificados de máquina para computadores Windows fora do domínio Active Directory, utilizando a infraestrutura de PKI corporativa baseada em Microsoft AD CS (MSAD-CA).
O processo utiliza CSR (Certificate Signing Request) gerado localmente e emissão via template dedicado na CA do domínio.

## Objetivo

Permitir que máquinas fora do domínio obtenham certificados digitais emitidos pela CA corporativa para uso em:
- 802.1X (NAC / Cisco ISE / NPS)
- VPN
- Autenticação mútua TLS
- Serviços internos
- Dispositivos em DMZ ou ambientes isolados

## Arquitetura

Componentes envolvidos:
- MSAD (Active Directory Domain Services)
- MSAD-CA (Active Directory Certificate Services)
- Template dedicado: Computer-External
- Máquina fora do domínio: WIN10

## Pré-requisitos

### 1) Template de certificado

Template configurado na CA com as seguintes características:
```
- Nome: Computer-External
- Baseado em: Computer ou Workstation Authentication
- Subject Name: Supply in the request
- EKU:
- Client Authentication
- Server Authentication
- Permissões:
- Grupo PKI-Admins → Read + Enroll
- Template devidamente publicado na CA.
```

### 2) Acesso à CA

Um dos métodos abaixo:
- A) Web Enrollment: http://MSAD-CA/certsrv
- B) certreq no servidor da CA

## Procedimento Operacional

### 1) Gerar CSR na máquina fora do domínio
### 1.1 Criar arquivo INF

Criar o arquivo machine.inf:
```
[Version]
Signature="$Windows NT$"

[NewRequest]
Subject = "CN=WIN10"
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
MachineKeySet = TRUE
RequestType = PKCS10
KeyUsage = 0xa0

[Extensions]
2.5.29.17 = "{text}"
_continue_ = "dns=WIN10"
_continue_ = "dns=WIN10.seudominio.local"
```

### 1.2 Gerar CSR

Executar no Prompt de Comando como Administrador:
```
certreq -new machine.inf machine.req
```
Resultado:
- Arquivo CSR gerado: machine.req

### 2) Emitir certificado na CA

Método A — Web Enrollment (padrão suporte)

1) Acessar:
```
http://MSAD-CA/certsrv
```

2) Selecionar:
- Request a certificate > Advanced certificate request > Submit a certificate request

3) Colar o conteúdo do arquivo machine.req

4) Selecionar o template:
  - Computer-External

5) Emitir e baixar o certificado em formato Base64 (.cer)

Método B — certreq no servidor da CA (requer acesso à CA)
```
certreq -submit -attrib "CertificateTemplate:Computer-External" machine.req machine.cer
```

### 3) Instalar cadeia de certificação (obrigatório)

Importar os certificados da CA:
Root CA
```
Local:
Local Computer → Trusted Root Certification Authorities
```
Intermediate CA (se aplicável)
```
Local:
Local Computer → Intermediate Certification Authorities
```

### 4) Instalar certificado na máquina WIN10

Copiar o arquivo machine.cer para a máquina WIN10 e executar:
```
certreq -accept machine.cer
```

### 5) Validação

Abrir:
```
certlm.msc
```

Verificar:
- Local: Computer → Personal → Certificates
- Subject: CN=WIN10
- Template: Computer-External
- Private Key: disponível
- EKU: Client Authentication

Comando adicional:
```
certutil -store my
```

## Observações Importantes

### Segurança PKI
- Não utilizar o template padrão "Computer" para máquinas fora do domínio.
- Utilizar sempre templates dedicados para dispositivos externos.
- Controlar permissões de Enroll.
- Registrar emissões via ticket/change.

### Boas práticas
- Definir validade menor para certificados externos (ex: 12 meses).
- Padronizar nomenclatura de CN e SAN.
- Manter inventário de certificados emitidos.
- Revisar periodicamente templates da CA.

### Referências
- Microsoft AD CS Documentation
- RFC 5280 – X.509 Public Key Infrastructure
- Cisco ISE / NPS Certificate Requirements
