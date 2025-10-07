# Flutter - Arquitetura Base

Este projeto serve como **modelo base** para o desenvolvimento de aplicações Flutter modulares, utilizando uma arquitetura escalável e organizada por **features**.

---

## Objetivo

Fornecer uma estrutura consistente e reutilizável para criação de novos apps ou microaplicações dentro de um mesmo ecossistema, com separação clara de responsabilidades e suporte a:

- Gerenciamento de estado com **BLoC**  
- Navegação com **GoRouter**  
- Injeção de dependências com **GetIt**  
- Controle de sessão persistente  
- Integração com notificações **Braze**  
- Redirecionamento via **Deeplink**

---

## Estrutura do Projeto

```
lib/
 ├── core/                     # Infraestrutura global
 │   ├── di/                   # Registros globais de dependências (GetIt)
 │   ├── router/               # Configuração do GoRouter e rotas globais
 │   ├── session/              # SessionManager, TokenStorage, AuthInterceptor
 │   ├── notifications/        # Braze handler, mapeamento de payloads
 │   └── network/              # Client HTTP, interceptors, endpoints base
 │
 ├── commons/                  # Componentes e utilitários reutilizáveis
 │   ├── widgets/              # Botões, inputs, loaders, etc.
 │   ├── themes/               # Cores, tipografia, estilos globais
 │   ├── formatters/           # Formatação de data, CPF, moeda, etc.
 │   └── extensions/           # Extensões utilitárias
 │
 ├── features/                 # Features independentes
 │   ├── login/
 │   │   ├── ui/               # Páginas, widgets, blocos
 │   │   ├── domain/           # Entidades, repositórios, usecases
 │   │   └── data/             # Models, datasources, implementações
 │   ├── onboard/
 │   ├── home/
 │   └── ...
 │
 └── main.dart                
```

---

## Camadas por feature

Cada feature segue o padrão:

```
feature_name/
 ├── ui/
 │   ├── pages/
 │   ├── components/
 │   └── bloc/
 ├── domain/
 │   ├── entities/
 │   ├── repositories/
 │   └── usecases/
 └── data/
     ├── datasources/
     ├── models/
     └── repositories/
```

Essa divisão garante **baixo acoplamento** e **alta coesão** entre as camadas.

---

## Boas Práticas

- **Não misturar** responsabilidades de UI com lógica de negócio.  
- **Evitar dependências cruzadas** entre features.  
- **Manter o core leve** (infra apenas, sem lógica de domínio).  
- **Commons só com componentes visuais** e helpers puros.  

---

## Não sei se faz sentido agora

- Modularização via packages internos (se necessário no futuro)  
- Internacionalização (i18n)

---

Arquitetura pensada para escalabilidade, manutenibilidade e simplicidade de evolução.
