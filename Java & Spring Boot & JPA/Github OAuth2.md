<h1>Github OAuth2 Login 구현</h1>

- 어떤 프로젝트에서 Github OAuth2 로그인 연동을 구현해야한 적이 있다.  
  인터넷에 나와있는 대부분의 코드들은 Thymeleaf 등의 STE를 사용하는 예제들이 많았는데,  
  아래 설명은 프론트엔드 코드는 ReactJS로 따로 있고, 백엔드 코드를 Spring으로 작성한 예시이다.

- 또한 ReactJS의 코드는 `localhost:3000`에서, 그리고 백엔드 코드는 `localhost:8080`에서 구동된다고 가정한다.

- 참고로 Github의 `OAuth Apps` 설정은 아래와 같다.

![picture 1](../images/96df25f1d82f1e1d6be6982aa6b380c17497947b9b3c780c56147a27bcc6480f.png)

<h2>Spring Security 도입하기</h2>

- Spring Security는 Google, Facebook, Github 등에 대한 OAuth2 지원을 기본적으로 해준다.
- 우선 Spring Security를 통해 OAuth를 사용하기 위한 의존성은 아래와 같다.

```gradle
dependencies {
    // Other dependencies..
    implementation('org.springframework.boot:spring-boot-starter-oauth2-client')
}
```

<hr/>

<h2>Spring Security Configuration</h2>

- 가장 먼저, Spring Security 설정 클래스를 작성해보자.  
  아래의 `SecurityConfig` 클래스가 설정 클래스이다.

```java
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final GithubOAuth2UserService githubOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        http
                .httpBasic().disable()
                .headers().frameOptions().disable()
                .and().csrf().disable()
                .cors().configurationSource(corsConfigurationSource())
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                    .authorizeRequests()
                        .antMatchers("/v1/**")
                        .permitAll()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .successHandler(authenticationSuccessHandler())
                        .failureHandler(authenticationFailureHandler())
                        .userInfoEndpoint()
                        .userService(githubOAuth2UserService);
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.addAllowedOriginPattern("*");
        configuration.addAllowedHeader("*");
        configuration.addAllowedMethod("*");
        configuration.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }

    @Bean
    public AuthenticationFailureHandler authenticationFailureHandler() {
        return new GithubOAuthExceptionHandler();
    }

    @Bean
    public AuthenticationSuccessHandler authenticationSuccessHandler() {
        return new GithubOAuthOnSuccessHandler();
    }
}
```

- 위 코드에 대한 설명은 아래와 같다.
  - `http.httpBasic().disable()` : HTTP 기본 인증 방식을 비활성화 한다.
  - `.headers().frameOptions().disable()` : 응답(Response) 헤더에 Security Header를 비활성화시킨다.
  - `.and().csrf().disable()` : CSRF를 비화성화한다.
  - `.cors().configurationSource(corsConfigurationSource())` : CORS에 사용되는 `CorsFilter`에  
    `corsConfigurationSource()` Bean을 등록한다.
  - `sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELSS)` : Stateless Server  
    Application임을 지정한다.
  - `and().authorizeRequests().antMatchers("/v1/**").permitAll()` : `/v1/**`의 패턴에 맞는  
    엔드포인트를 인증 없이 사용할 수 있도록 한다.
  - `and().logout().logoutSuccessUrl("/")` : 로그아웃 성공 url을 지정한다.
  - `and().oauth2Login().successHandler(authenticationSuccessHandler())` : oauth2 로그인이 성공했을 때의  
    작업을 처리할 핸들러를 지정한다.
  - `.failureHandler(authenticationFailureHandler())` : oauth2 로그인 실패 시 예외를 처리할 핸들러를 지정한다.
  - `.userInfoEndpoint().userService(githubOAuth2UserService)` : OAuth2를 사용하여 사용자 정보를 받아온 후  
    작업을 처리할 서비스 클래스를 지정한다.

<hr/>

<h2>OAuthAttributes</h2>

- <h2>Service</h2>
  <h2>Exception Handling</h2>
  <h2>Handlers</h2>