## Descrição
Este README documenta, passo a passo, o processo de criação de uma Máquina Virtual (VM) **Windows** pelo **Portal do Azure**. O objetivo é fornecer instruções reprodutíveis e seguras — do provisionamento à verificação e limpeza dos recursos 

---

## Pré-requisitos
- Conta ativa no [Azure Portal](https://portal.azure.com) com permissão para criar recursos (assinatura com quota suficiente).  
- Acesso ao portal com credenciais que permitam criar *Resource Groups*, *Virtual Networks*, *Public IPs* e *Virtual Machines*.  
- Cliente RDP disponível (Windows: `mstsc`, macOS: Microsoft Remote Desktop, Linux: `remmina`/`rdesktop`) caso vá conectar via IP público.  

> [!WARNING]
> esteja consciente de custos. VMs, endereços IP públicos, discos e serviços auxiliares geram cobrança. Para ambientes de laboratório, escolha tamanhos e discos de baixo custo e lembre-se de excluir o grupo de recursos ao final.  

---

## Passo a passo Criar VM no Portal do Azure

> [!NOTE]
> o fluxo abaixo segue a experiência padrão do **Azure Portal** (blade "Virtual machines"). Os nomes de campos ou posições de botões podem variar levemente com atualizações do portal; os conceitos permanecem os mesmos.

### 1. Abrir o Azure Portal
1. Acesse `https://portal.azure.com` e autentique-se com sua conta.

### 2. Criar recurso — Virtual machine
1. No menu lateral esquerdo clique em **Create a resource** → pesquise por **Virtual Machine** → clique em **Create** → **Virtual machine**.
2. Você será direcionado à tela **Create a virtual machine**.

### 3. Aba *Basics* (Preenchimento mínimo)
1. **Subscription:** selecione a assinatura desejada[^1].  
2. **Resource group:**  
   - Escolha um grupo existente ou **Create new**.  
   - Recomendo um nome descritivo: `rg-lab-vm-windows-<sigla>`.  
3. **Virtual machine name:** por exemplo `vm-windows-lab-01`.  
4. **Region:** escolha a região (ex.: `Brazil South`) mais próxima ao usuário/serviço[^2].  
5. **Availability options:** selecione conforme necessidade (No infrastructure redundancy required para laboratório).  
6. **Image:** escolha a imagem Windows desejada (ex.: **Windows Server 2022 Datacenter** ou **Windows 11 Pro** se for desktop).[^13]  
7. **Size:** clique em **Change size** e selecione um SKU econômico para testes (ex.: `Standard B2s` ou equivalente). Confirme[^3].  
8. **Administrator account:** defina `Username` e `Password` (ou configure uma autenticação por chave se a imagem suportar — tipicamente Windows usa senha)[^4].  
9. **Inbound port rules:** escolha *Allow selected ports* e selecione **RDP (3389)** somente se for necessário. Recomenda-se não abrir RDP ao mundo[^5].

[^1]: Algumas imagens e tamanhos de VM podem não estar disponíveis em todas as assinaturas. Verifique se sua assinatura tem acesso às imagens desejadas e quota suficiente para criação de recursos.
[^2]: A escolha da região afeta latência, disponibilidade de recursos e custos. Regiões como Brazil South podem ter preços diferentes de East US ou West Europe.
[^3]: SKUs como Standard B2s são ideais para testes e laboratórios, mas não recomendados para cargas de produção. Consulte a documentação oficial de tamanhos para detalhes.
[^4]: O Azure exige senhas fortes para contas de administrador. Evite senhas triviais como admin123 — utilize combinações com letras maiúsculas, minúsculas, números e símbolos.
[^5]: Expor a VM com IP público e RDP aberto para qualquer origem (Any) é altamente desaconselhado. Utilize NSGs com regras restritivas ou Azure Bastion para acesso seguro.
[^6]: Essa funcionalidade é útil para evitar cobranças fora do horário de uso. Pode ser configurada com notificações via e-mail antes do desligamento.
[^7]: Recurso pago que permite acesso RDP/SSH via navegador, sem expor portas públicas. Ideal para ambientes corporativos e de produção.
[^8]: Permite executar comandos diretamente na VM sem precisar de acesso remoto. Útil para troubleshooting ou verificação rápida de configurações.
[^9]: São metadados que ajudam na organização e rastreamento de recursos. Podem ser usadas para aplicar políticas, gerar relatórios de custo ou facilitar automações.
[^10]: Essa ação é irreversível. Todos os recursos dentro do grupo serão apagados, incluindo discos que possam conter dados importantes.
[^11]: Embora o foco seja o Portal, o uso da CLI permite automações e integrações com scripts. Ideal para DevOps e ambientes repetitivos.
[^12]: Recursos como discos premium, IPs públicos e diagnósticos avançados geram cobranças adicionais. Use a [calculadora de preços do Azure](https://azure.microsoft.com/pricing/calculator/) para estimativas.
[^13]: Algumas imagens, como Windows 11, podem exigir licenciamento adicional ou configurações específicas de hardware virtualizado (ex.: TPM virtual).
[^14]: Ativar essas opções pode ajudar na observabilidade da VM, mas também aumenta o custo. Avalie conforme o objetivo do ambiente.

### 4. Aba *Disks*
1. **OS disk type:** selecione **Standard SSD** ou **Premium SSD** conforme necessidade de desempenho/custo[^12].  
2. (Opcional) Adicione discos de dados se precisar de armazenamento adicional.  
3. Configure encriptação e backup conforme política da organização.

### 5. Aba *Networking*
1. **Virtual network (VNet):** selecione uma VNet existente ou crie uma nova (ex.: `vnet-lab-01`).  
2. **Subnet:** escolha a subnet (ex.: `subnet-1`).  
3. **Public IP:** selecione **Create new** se desejar acessibilidade via IP público (ex.: `pip-vm-windows-01`) OU selecione **None** para não expor a VM publicamente[^5].  
4. **NIC network security group (firewall):**  
   - **Basic**: selecione **Allow selected ports** e mantenha apenas o necessário.  
   - **Advanced**: prefira usar NSG (Network Security Group) com regras restritivas (permitir RDP somente de IPs confiáveis).  

> [!WARNING]
> abrir RDP (porta 3389) para `Any`/`Internet` expõe fortemente a VM a ataques. Se precisar de acesso remoto, prefira Azure Bastion ou NSG que restrinja origem por CIDR/IP.

### 6. Aba *Management*
1. **Boot diagnostics:** habilite (útil para troubleshooting)[^6].  
2. **OS guest diagnostics / Monitoring:** habilite conforme necessidade. (Aumenta custo se ativar diagnósticos persistentes/insights)[^14].  
3. **Auto-shutdown:** configure horário de desligamento automático para economizar custos (altamente recomendado em laboratórios)[^6].  
4. **Backup:** configure apenas se necessário[^12].

### 7. Aba *Advanced* (opcional)
1. Configure extensões de VM (ex.: VM Agent, scripts) se precisar executar custom scripts pós-criação.
> [!TIP]
> Embora mais comuns em Linux, algumas imagens Windows também suportam extensões para automação pós-provisionamento.
2. Defina parâmetros de provisionamento e cloud-init (quando aplicável).

### 8. Aba *Tags*
1. Adicione tags úteis para organização (ex.: `env:lab`, `owner:devops`, `project:exercicio-vm`)[^9].

### 9. Review + Create
1. Clique em **Review + create**. O Azure fará validação dos campos.  
2. Revise custos e configurações[^12].  
3. Clique em **Create**. Aguarde o *deployment* até o status **Deployment succeeded**.

---

## Verificação

### 1. Conferir no Portal
1. No menu lateral vá em **Virtual machines** → selecione a VM criada (`vm-windows-lab-01`).  
2. Verifique o **Status**: deve estar `Running`.  
3. Na visão geral (Overview) confira:
   - **Public IP address** (se configurado)  
   - **Private IP**  
   - **Size**, **OS disk**, e **Resource group**

### 2. Conexão via RDP (quando public IP disponível)
1. Obtenha o **Public IP** na página Overview.  
2. Abra o cliente RDP (Windows: `mstsc`) e conecte para `PUBLIC_IP:3389`.  
3. Use as credenciais configuradas (Username e Password).  
4. Ao conectar, confirme se consegue abrir o **Server Manager** (Windows Server) ou **Settings > System** (Windows Desktop).  

### 3. Alternativa mais segura: Azure Bastion
1. Em vez de expor RDP publicamente, crie **Azure Bastion** no VNet e conecte via Portal → *Connect* → *Bastion*[^7].  
2. Bastion evita expor porta 3389 na internet e é recomendação para ambientes produtivos.

### 4. Verificação adicional (via Portal)
1. Use **Run command** (VM > Operations > Run command) para executar comandos remotos (ex.: `systeminfo` ou `winver`) e validar OS e patches[^8].  
2. Verifique métricas básicas (CPU, Network In/Out) no blade **Monitoring**[^14].

---

## Limpeza de recursos
Para evitar cobranças, remova todos os recursos relacionados quando não precisar mais.

### Excluir o Resource Group (recomendado)
1. No menu esquerdo clique em **Resource groups**.  
2. Selecione o resource group usado durante criação (ex.: `rg-lab-vm-windows-<sigla>`).  
3. Clique em **Delete resource group**.  
4. Digite o nome do resource group exatamente para confirmar e clique em **Delete**[^10].  
   - Isso removerá a VM, discos, IPs públicos, NICs e outros recursos contidos.  
5. Verifique no portal depois que o processo terminar (pode levar alguns minutos).

> [!WARNING]
> excluir o resource group apaga permanentemente todos os recursos dentro dele. Certifique-se de fazer backup de dados importantes antes de proceder.

---

## Apêndice (opcional): Exemplo rápido com Azure CLI
Se preferir automatizar via `az cli`, um exemplo mínimo (apenas referência — o usuário pediu fluxo no Portal):

```bash
# Login
az login

# Variáveis de exemplo
RG="rg-lab-vm-windows"
LOCATION="brazilsouth"
VM_NAME="vm-windows-lab-01"
ADMIN_USER="admin"
ADMIN_PASS="P@ssw0rdExample!"   # substitua por senha forte
IMAGE="MicrosoftWindowsServer:WindowsServer:2022-Datacenter:latest"
SIZE="Standard_B2s"

# Criar resource group
az group create -n $RG -l $LOCATION

# Criar VM (gera VNet, Subnet, Public IP, NIC automaticamente)
az vm create \
  -g $RG -n $VM_NAME \
  --image $IMAGE \
  --admin-username $ADMIN_USER \
  --admin-password $ADMIN_PASS \
  --size $SIZE \
  --public-ip-sku Standard \
  --authentication-type password
