# Configurando ambiente de testes em Java com Banco de dados H2 e JPA

Suponhamos que você tenha uma classe Usuário que foi criada utilizando as anotações provenientes do JPA para mapeamento e criação dessa entidade no banco de dados.

```
@Entity
@Table(name = UsuarioDbConstantes.TABLE_NAME)
public class Usuario implements UserDetails {

    @Column
    private String email;
    @Column
    private String password;
    
 }
```

Você terá um repository que contém métodos que serão implementados no seu service com suas regras de negócio para essa entidade Usuário.

```
@Service
public class UsuarioService {

@Autowired
private UsuarioRepository usuarioRepository;

/*
* Busca usuário pelo id
* @param id Id do usuário
*/
public Usuario findById(Long id) {
	final Optional<Usuario> usuarioEncontrado = usuarioRepository.findById(id);
    if (usuarioEncontrado.isEmpty()) {
        throw new UsuarioException("Usuário não encontrado.");
    }
    return usuarioEncontrado.get();
}
 
}
```

Sua regra foi criada e para validar o funcionamento do método e garantir que seu comportamento não mude posteriormente, você deseja criar testes para esse método. 

Com isso, será necessário configurar o ambiente de desenvolvimento para executar os testes. Nesse caso, pode ser utilizadada a abordagem da API Mockito para testes, onde você pode "mockar" o repository da entidade na classe de testes e injetar o mock no service que estará sendo testado. Ou é possível integrar sua aplicação em um banco em memória, utilizando o banco H2, que facilita testes de integração simulando a estrutura do seu banco de produção. 

Para isso, criei esse tutorial para facilitar o entendimento de como configurar essa integração entre o banco de dados H2 em uma aplicação Java, tendo em vista meu interesse em aprender mais sobre a configuração dos tipos de ambiente de de uma aplicação real, ou seja, ambiente de desenvolvimento e ambiente de testes,

- Incluir dependência no arquivo pom.xml:

```
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

- Criar na pasta de testes o arquivo application.properties e adicionar as seguintes configs:

  ```
  spring.datasource.driver-class-name=org.h2.Driver
  spring.datasource.username=
  spring.datasource.password=
  spring.jpa.open-in-view=false
  spring.jpa.properties.hibernate.show_sql=true
  
  # Perfil ativo
  spring.profiles.active=test
  # Se utiliza flyway no ambiente dev, setar para false para não executar migrations
  spring.flyway.enabled=false
  ```

  - Configurar classe de configuração da injeção de dependências:

  ```
  @Configuration
  @Profile("test")
  public class TestConfig {
  
      @MockBean
      private AuthenticationManager authenticationManager;
  }
  ```

  *Como no projeto no qual eu estava trabalhando estava sendo utilizada uma dependência do SpringSecurity, tive que injetar a classe AutenticationManager. Porém, serve de exemplo para casos onde seu ambiente de testes precisará de um bean que deve ser injetado na inicialização do conteiner*

  

  ## Criar classe de Testes

  Sua classe de testes precisará das seguintes anotações:

  ```
  @SpringBootTest
  @RunWith(SpringRunner.class)
  ```

  Pronto!

  Agora basta você injetar seu repository, service ou qualquer outra classe utilizando construtor, @Autowired ou método setter para que você possa realizar operações de persistência com suas entidades criadas utilizando o JPA!

  
