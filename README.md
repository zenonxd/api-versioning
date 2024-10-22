<p align="center">
  <img src="https://img.shields.io/static/v1?label=Spring Essential - Dev Superior&message=Versionamento de API&color=8257E5&labelColor=000000" alt="envio de email - sendgrid" />
</p>

# Objetivo

Veremos duas abordagens diferentes de versionamento de API RST, mostrando as suas principáis características.

1. Versionamento via URI
2. Versionamento via Media Type

# Por quê versionar uma API REST

O primeiro motivo é com relação ao formato de dados. O formato define como os dados serão trocados entre um web server 
e um cliente.

Seja quando acessamos determinado site ou requisitamos alguma informação de uma API, ela vai trocar os dados conosco.

Esses dados podem ser definidos em alguns formatos, como JSON e XML.

Porém, a ideia é que seja disponibilizado nos dois. 

## XML

![img.png](img.png)

Para um serviço mais legado que permanecerá disponível.

## JSON

E disponibilizar também endpoints para serem consultados no formato JSON.

![img_1.png](img_1.png)

Logo, para termos ambos formatos, precisamos **versionar** a nossa API REST.

Mas podemos ter outro motivo também! Os dados podem ter uma mudança significativa por um determinado endpoint:

![img_2.png](img_2.png)

No exemplo acima, o JSON possui a propriedade salary! Podemos considerar o lado esquerdo como uma Versão 0 (padrão) e a
do lado direito como uma Versão 1.0.

# Versionamento via URI

A versão é indicada na própria URI (A URI direciona para as versões específicas)

![img_3.png](img_3.png)

Esse "v1", muda quando houver mudanças significativas nos dados da aplicação. 

![img_4.png](img_4.png)

As duas versões podem ser acessadas!

## Características

- Abordagem prática e simples de se implementar
- Quando a versão é incluída no espaço da URI, a representação do recurso é considerada imutável
- Não é recomendado que a versão seja mantida por um longo tempo
- Existe uma polêmica no uso desta abordagem

Muitas pessoas não recomendam versionamento via URI, pelo motivo 2 e 3.

# Versionamento via Media Type

Neste caso, a versão do recurso é indicada no cabeçalho da requisição. Também chamado de **versão de representação do 
recurso.**

Nesse ".name" do accept pode ser: nome da empresa/projeto.

![img_5.png](img_5.png)

## Características

- A URi não muda. O que fazemos é especificar um media type customizado
- Nesta abordagem o cliente não faz nenhuma suposição em relação à estrutura do response (além do que é definido no
media type -a header-)
- Não fornece informações semânticas suficientes e necessita que o cliente forneça informações adicionais no cabeçalho.

# Prática - Via URI

Tenha em mente essa requisição do Postman do projeto DSMovie.

![img_6.png](img_6.png)

Imagine que a gente queira incluir o campo "gênero" nesse retorno, porém, mantendo a URL (tendo duas consultas na 
mesma URI). Faremos o versionamento de API.

## Entity

Iremos nas nossas entities (Movie) e criaremos um campo "gênero". Para isso, criaremos a entidade Genre.

```java
@Entity
@Table(name = "tb_genre")
public class Genre {
    
    @Id
    @GenerateValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    //construtor com e sem argumento + getters and setters
}
```
E agora lembramos do relacionamento, né? Um gênero pode ter vários filmes (Set). Em resumo, um OneToMany!

Movie recebe Genre com @ManyToOne e @JoinColumn.

Genre recebe Movie com @OneToMany e mappedBy.

Lembrar dos Getters e Setters.

## DTO

Agora precisamos ir para o DTO, visto que ele ainda não possui o campo Genre. A ideia não é mexer no MovieDTO já
existente, e sim criar um novo, com os mesmos atributos + genre!

Criaremos um MovieGenreDTO (lembrar de colocar também no construtor que troca de Entity para DTO):

## Controller

Agora sim começa o versionamento. Nós manteremos o endpoint "/movies". 

Clonaremos a classe MovieController. Ela chamará "MovieControllerV1".

Agora o findAll não retornará mais **MovieDTO** e sim **MovieGenreDTO**.

## Service

Diferente do Controller, não precisa criar um novo Service. Podemos simplesmente copiar a lógica do método findAll e
fazer as alterações.

![img_7.png](img_7.png)

Podemos fazer a mesma coisa para o findById.

![img_8.png](img_8.png)

## Voltando ao Controller

Agora na nova classe criada não será "/movies" e sim "/v1/movies". 

![img_9.png](img_9.png)

# Prática - Via Media Type

![img_11.png](img_11.png)

Agora nós vamos passar no cabeçalho um "accept:application/vdn.name.v1+tipo do dado"

## Controller

Vamos no MovieController e dar um Ctrl C + Ctrl V no método findAll.

No novo método faremos o seguinte:

1. O seu nome será findAllV1 
2. Retornará um MovieGenreDTO ao invés de MovieDTO
3. O service não usará mais o findAll e sim o findAllMovieGenre (criado na etapa de URI)

![img_10.png](img_10.png)

## Ok, e para inserir o cabeçalho?

Passaremos no GET do método no Controller.

![img_12.png](img_12.png)

Mesma coisa no findById!

![img_13.png](img_13.png)

## Postman

Passamos no postman a URI padrão "/movies", porém em headers criaremos um campo "Accept" com o código passado dentro do
produces:

![img_14.png](img_14.png)

Com isso, ele trará o gênero.

Mesma coisa na requisição do findById, só criar uma header com o código do produces.