## Na classe SecurityConfig, registrar componente (Interface):

@Bean
public PasswordEncoder getPasswordEncoder() {
    return new BCryptPasswordEncoder();
}

## Ao salvar password do usuário, usar a seguinte lógica

    @Autowired
    private PasswordEncoder passwordEncoder

    password = passwordEncoder.encode(user.password);

