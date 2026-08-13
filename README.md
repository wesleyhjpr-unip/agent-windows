# Guia de Configuração do Agente Windows para Azure DevOps

Este guia descreve, passo a passo, como configurar um **agente auto-hospedado** do Azure DevOps em uma máquina Windows, incluindo a criação do token de acesso pessoal (PAT) e do *agent pool*.

## Índice

1. [Pré-requisitos](#pré-requisitos)
2. [Criar um Token de Acesso Pessoal (PAT)](#1-criar-um-token-de-acesso-pessoal-pat)
3. [Criar um Agent Pool](#2-criar-um-agent-pool)
4. [Configurar o Agente na Máquina Windows](#3-configurar-o-agente-na-máquina-windows)

## Pré-requisitos

- Acesso à organização no Azure DevOps (`https://dev.azure.com/unipti`).
- Conta com permissão para criar tokens e pools de agentes.
- Máquina Windows (Windows 10/11 ou Windows Server) onde o agente será instalado.
- O **PAT** será fornecido pelo Emerson (necessário para autenticar o agente).

---

## 1. Criar um Token de Acesso Pessoal (PAT)

O PAT é um mecanismo de autenticação alternativo ao Azure DevOps, utilizado para substituir a senha em ferramentas externas, como o agente de build.

> 📖 **Referência:** [Documentação oficial sobre Tokens de Acesso Pessoal](https://learn.microsoft.com/pt-br/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows)

1. Acesse sua organização em [https://dev.azure.com/unipti](https://dev.azure.com/unipti).

2. No canto superior direito, abra as **Configurações do usuário** e selecione **Tokens de acesso pessoal**.

   ![](img/1.png)
   ![](img/2.png)

3. Clique em **+ New Token** para criar um novo token.

   ![](img/3.png)

4. Dê um nome ao token (por exemplo, `IIS-homologacao-token`), selecione a organização onde ele será usado e defina uma data de expiração automática.

   ![](img/4.png)

5. Ao finalizar, **copie o token e guarde-o em um local seguro**. Por segurança, ele **não será exibido novamente**.

   ![](img/5.png)

---

## 2. Criar um Agent Pool

O *agent pool* é o grupo de agentes que executará os pipelines. Siga os passos abaixo para criá-lo:

1. Abra um navegador e acesse as **Configurações da Organização**:

   A. Entre em sua organização em [https://dev.azure.com/unipti](https://dev.azure.com/unipti).

   B. Selecione as **Configurações da Organização** no menu lateral esquerdo.

   ![](img/6.png)

   C. Acesse **Pools de Agentes** e clique em **Add pool**.

   ![](img/7.png)

   D. Selecione **Self-hosted**, defina um nome (por exemplo, `IIS-homologacao`) e clique em **Create**.

   ![](img/8.png)

---

## 3. Configurar o Agente na Máquina Windows
> 📖 **Referência:** [Documentação oficial sobre configuração de agentes do Windows auto-hospedados](https://learn.microsoft.com/pt-br/azure/devops/pipelines/agents/windows-agent?view=azure-devops&tabs=IP-V4)

> ⚠️ Esta etapa deve ser executada **no servidor** onde o agente será instalado.

1. Na página do pool criado, clique em **New agent**.

   ![](img/9.png)

### 3.1. Baixar o agente

Clique em **Download** na tela do Azure DevOps ou acesse diretamente o [link de download do agente](https://download.agent.dev.azure.com/agent/5.277.0/vsts-agent-win-x64-5.277.0.zip).

Salve o arquivo `.zip` na pasta `Downloads` (ou no local de sua preferência).

### 3.2. Extrair o agente

Crie a pasta de instalação e extraia o conteúdo do arquivo baixado:

```powershell
PS C:\> mkdir agent ; cd agent
PS C:\agent> Add-Type -AssemblyName System.IO.Compression.FileSystem ;
[System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-5.277.0.zip", "$PWD")
```

### 3.3. Configurar o agente

Execute o script de configuração:

```powershell
PS C:\agent> .\config.cmd
```

Siga as instruções exibidas no terminal e informe:

- **URL da organização** — `https://dev.azure.com/unipti`;
- **Tipo de autenticação** — escolha **PAT** e informe o token fornecido pelo **Emerson**;
- **Nome do pool** e **nome do agente** — use `IIS-homologacao`.

Quando a pergunta **"Enter run agent as service? (Y/N)"** aparecer, responda com **Y** (sim). O script solicitará as credenciais da conta de serviço usada para executar o agente.

Após a reinicialização da máquina, o serviço do agente deve iniciar automaticamente e o agente deve aparecer como **Online** no Azure DevOps.

![](img/10.png)

> 💡 **Dica importante sobre a conta de serviço**
>
> É uma prática de segurança recomendada usar uma **conta de serviço dedicada com privilégios mínimos** (como `NT AUTHORITY\NETWORK SERVICE`) para executar o agente, em vez de uma conta de administrador. Durante a configuração interativa, você pode especificar essa conta.
>
> Se o agente não iniciar automaticamente após esses passos, pode ser necessário remover e reconfigurar completamente o agente.
