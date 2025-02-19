spring:
  ldap:
    urls: ldap://your-ldap-server:389
    base: DC=OLKYPAY,DC=LOCAL
    username: CN=ldap_admin,DC=OLKYPAY,DC=LOCAL  # Admin user for LDAP lookup
    password: your_admin_password
    user-dn-patterns: "CN={0},OU=OlkyWallet,OU=ServicesAccounts,OU=Groupe_Olky,DC=OLKYPAY,DC=LOCAL"
    group-search-base: "OU=OlkyWallet,OU=ServicesAccounts,OU=Groupe_Olky"
    group-search-filter: "(member={0})"