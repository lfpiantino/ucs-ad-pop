# ucs-ad-pop
Procedimento Operacional Padrão (POP) para implantação de um controlador de domínio com UCS (Univention Corporate Server) e Samba4.

POP - Implantação de Controlador de Domínio com UCS (Samba4)
Projeto: Implantação UCS + Active Directory
Ambiente: Teste / Pré-Produção
Responsável: Luiz Fernando Piantino
Data: [Atualizar com data real]

1. Instalação do UCS (Univention Corporate Server)
- Versão utilizada: UCS 5.2 baseada em Debian 12 (Bookworm)
- Modo de instalação: Primary Directory Node
- Nome do host inicial: ucs-9641
- Domínio FQDN: lfpiantino.intranet
- NetBIOS (workgroup): LFPIANTINO

2. Configurações Iniciais
- Atualização do sistema: univention-upgrade
- Verificação de scripts pendentes:
  univention-check-join-status
  univention-run-join-scripts
- Instalação e ativação dos módulos:
  - univention-s4-connector
  - univention-samba4
  - univention-admin-diary-backend (pendente mas irrelevante para AD)

3. Correções e Ajustes
DNS:
- Verificação do serviço: systemctl status named
- Ajuste no firewall:
  ufw allow 53/tcp
  ufw allow 53/udp
  ufw reload
- Ajustes em /etc/bind/named.conf.options:
  options {
      directory "/var/cache/bind";
      recursion yes;
      allow-query { any; };
      listen-on { any; };
      dnssec-validation no;
  };

NTP (sincronização de hora):
apt install ntp -y
systemctl restart ntp

Verificação de portas abertas:
ss -tuln | grep :53

4. Testes de SRV no Windows:
nslookup -type=SRV _ldap._tcp.lfpiantino.intranet
nslookup -type=SRV _kerberos._tcp.lfpiantino.intranet

5. Ajustes no Samba4
- Validação: systemctl status samba-ad-dc
- Ajuste do nome NetBIOS (opcional): ucr set samba/domain=LFPIANTINO

6. Ingresso da máquina Windows
- IP DNS apontado para: 192.168.41.18
- Sincronização de hora no Windows: w32tm /resync
- Ingresso com:
  - Domínio: LFPIANTINO
  - Usuário: Administrator
  - Senha: [senha do UCS]

7. Alteração do nome do host (opcional)
ucr set hostname=dc01
hostnamectl set-hostname dc01.lfpiantino.intranet
univention-system-setup --force
reboot

Resultado Final:
- Máquina ingressada com sucesso
- Controlador de domínio UCS operando como AD-DC
- DNS funcional
- NetBIOS nome personalizado
- Ambiente pronto para novos testes e produção

