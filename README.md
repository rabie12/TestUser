To configure Spring Security Basic Authentication and then retrieve groups and roles from LDAP, follow these steps:

✅ 1. Define LDAP Properties in application.yml

Configure the LDAP connection:

spring:
  ldap:
    urls: ldap://your-ldap-server:389
    base: DC=OLKYPAY,DC=LOCAL
    username: CN=ldap_admin,DC=OLKYPAY,DC=LOCAL  # Admin user for LDAP lookup
    password: your_admin_password
    user-dn-patterns: "CN={0},OU=OlkyWallet,OU=ServicesAccounts,OU=Groupe_Olky,DC=OLKYPAY,DC=LOCAL"
    group-search-base: "OU=OlkyWallet,OU=ServicesAccounts,OU=Groupe_Olky"
    group-search-filter: "(member={0})"

✅ This config:
	•	Connects to the LDAP server.
	•	Authenticates users via Basic Authentication.
	•	Searches for user groups under OU=OlkyWallet.

✅ 2. Configure Security (SecurityConfig.java)

Enable Basic Authentication and integrate LDAP role fetching.

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
            .httpBasic() //