# Cenário
Cisco Identity Services Engine (ISE) está joinado ao Domínio A

- Existe trust entre:
  - Domínio A
  - Domínio B
- Usuário: userB@dominioB.local
- Método: 802.1X (EAP-PEAP ou EAP-TLS)

## Como o ISE Autentica no AD
Quando o ISE é joinado ao AD:
- Ele cria um computer account no domínio
- Usa Kerberos e LDAP
- Se integra via API do AD (não é um DC, mas age como membro)

ISE usa o Domain Controller do domínio ao qual está associado para autenticação.

## Fluxo de Autenticação 802.1X (Usuário do Domínio B)
Etapas:
1. Cliente envia credenciais (userB@dominioB.local)
2. Switch/WLC encaminha para ISE via RADIUS
3. ISE consulta o AD (Domínio A)

Aqui entra a trust.

4. O DC do Domínio A:
   - Reconhece que o sufixo pertence ao Domínio B
   - Usa a trust
   - Encaminha a autenticação para um DC do Domínio B
5. DC do Domínio B valida a senha
6. Resultado retorna:
   - DC B → DC A
   - DC A → ISE
7. ISE aplica políticas de autorização
   - Isso é autenticação cross-domain via trust.

## Condições Necessárias Para isso funcionar
1. Trust deve estar operacional
   - DNS resolvendo ambos os domínios
   - Porta 88 (Kerberos) liberada
   - Porta 389/636 (LDAP)
   - RPC aberto

2️. Trust deve permitir autenticação
   - Não pode estar restrita com selective auth sem permissão
   - SID filtering não deve bloquear

3️. ISE deve:
   - Estar configurado para aceitar o sufixo do domínio B
   - Ter "User Principal Name (UPN)" corretamente tratado

## Atenção Importante
Existe diferença entre:
- Trust dentro da mesma floresta
  - Funciona de forma transparente.
- Trust entre florestas diferentes
  - Pode exigir:
     - Configuração de Name Suffix Routing
     - Ajuste de authentication scope
     - Configuração explícita no ISE

## Caso EAP-TLS (certificado)
Se for EAP-TLS:
 - ISE valida certificado
 - Pode mapear CN ou SAN para conta no AD
 - O lookup do usuário também seguirá a trust
Mas nesse caso o foco é mais:
 - Mapeamento de identidade
 - Validação de cadeia de certificação

## Problemas comuns em campo
 - DNS entre domínios não resolve
 - SPN incorreto
 - Trust unidirecional errada
 - Firewall bloqueando Kerberos referral
 - Selective authentication habilitada
