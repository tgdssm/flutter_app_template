# Flutter - Arquitetura Base

Este projeto serve como **modelo base** para o desenvolvimento de aplicações Flutter modulares, utilizando uma arquitetura escalável e organizada por **features**.

---

## Objetivo

Fornecer uma estrutura consistente e reutilizável para criação de novos apps ou microaplicações dentro de um mesmo ecossistema, com separação clara de responsabilidades e suporte a:

- Gerenciamento de estado com **Riverpod**
- Navegação com **GoRouter**
- Injeção de dependências via **providers** (sem service locator externo)
- Controle de sessão persistente com refresh automático
- Redirecionamento via **Deeplink**
- Monitoramento de erros com **Firebase Crashlytics**

---

## Estrutura do Projeto

```
lib/
 ├── app/                      # Composição: router, shell
 ├── core/                     # Implementações de infraestrutura global
 ├── shared/                   # Contratos, abstrações e modelos compartilhados
 ├── commons/                  # Componentes visuais e utilitários reutilizáveis
 ├── features/                 # Features independentes
 └── main.dart                 # Entry point com ProviderScope
```

---

## Camadas globais

### `app/`
Camada de composição — único lugar que conhece features, core e commons simultaneamente.

```
app/
 ├── router/                   # GoRouter com redirect auth guard
 └── ui/                       # Shell: tab bar, wrappers estruturais
```

### `core/`
Implementações concretas de serviços de infraestrutura. Sem lógica de domínio, sem conhecimento de features.

```
core/
 ├── config/                   # Ambientes, endpoints e configuração de API
 ├── monitoring/               # FirebaseErrorReporter
 ├── network/                  # DioHttpClientService, interceptors, ApiException
 │   └── network_providers.dart
 ├── notifications/
 ├── session/                  # SessionManagerImpl
 │   └── session_providers.dart
 └── storage/
     ├── flutter_secure_storage_impl.dart
     ├── token_storage.dart
     └── storage_providers.dart
```

### `shared/`
Interfaces, contratos e modelos compartilhados. Sem implementações concretas.

```
shared/
 ├── errors/                  # Result<T>, AppError, AppException
 ├── models/                  # Modelos compartilhados entre features
 ├── monitoring/              # Interface ErrorReporter
 ├── network/                 # Interface HttpClientService
 ├── notifications/           # Contratos de push e deeplink
 ├── router/                  # Interface AppRouter, Routes, Args
 ├── session/                 # Interface SessionManager
 └── storage/                 # Interfaces SecureStorage e SharedPreferences
```

### `commons/`
Componentes visuais e helpers sem lógica de negócio.

```
commons/
 ├── assets/
 ├── extensions/
 ├── formatters/
 ├── helpers/
 ├── strings/
 ├── typography/
 └── widgets/
     └── *.dart               # Componentes genéricos reutilizáveis
```

---

## Providers de infraestrutura

Cada módulo de `core/` declara seus próprios providers junto à implementação. Não há um arquivo central de bindings — cada provider conhece apenas suas dependências diretas via `ref`:

```dart
// core/network/network_providers.dart
final httpClientProvider = Provider<HttpClientService>((ref) {
  return DioHttpClientService();
});


```

Features consomem providers de core diretamente via `ref`:

```dart
// features/orders/orders_providers.dart
final ordersDatasourceProvider = Provider<OrdersDatasource>((ref) {
  return OrdersDatasourceImpl(ref.read(httpClientProvider));
});

final ordersRepositoryProvider = Provider<OrdersRepository>((ref) {
  return OrdersRepositoryImpl(ref.read(ordersDatasourceProvider));
});
```

O fluxo segue uma única direção:

```
core providers → feature providers → notifiers → UI

```
---

## Camadas por feature

```
feature_name/
 ├── feature_providers.dart       # Todos os providers da feature (DI + estado)
 ├── feature_routes.dart          # Rotas GoRoute da feature
 │
 ├── data/
 │   ├── datasources/
 │   │   ├── *_datasource.dart
 │   │   └── impl/*_datasource_impl.dart
 │   ├── models/
 │   └── repositories/
 │       └── *_repository_impl.dart
 │
 ├── domain/
 │   ├── repositories/
 │   │   └── *_repository.dart
 │   └── params/
 │
 └── ui/
     ├── notifiers/               # *Notifier, *State
     ├── pages/
     └── widgets/
```

---

## Padrões arquiteturais

### Result Pattern

Repositórios retornam `Result<T>` — sem exceções chegando à UI.

```dart
sealed class Result<T> {
  factory Result.success(T data) = Success<T>;
  factory Result.error(AppError error) = Error<T>;
}
```

### Tratamento de erros

```dart
// datasource — reporta e relança
} catch (error, stack) {
  ErrorReporter.instance.recordError(error, stack);
  throw AppException(AppError(type: ErrorType.unknown, ...));
}

// repository — converte para Result
} on AppException catch (e) {
  return Result.error(e.error);
}

// notifier — emite estado de erro
state = AsyncError(result.error!, StackTrace.current);
```

---

## Boas práticas

- **Não misturar** responsabilidades de UI com lógica de negócio
- **Evitar dependências cruzadas** entre features — modelos compartilhados ficam em `shared/models/`
- **`core/` sem conhecimento de features** — providers de infraestrutura só dependem de outros providers de core
- **`commons/` apenas** com componentes visuais e helpers puros
- **`shared/` apenas** com interfaces e modelos — sem implementações concretas
- **Providers de feature** declarados em `feature_providers.dart` — facilita localizar dependências

---
