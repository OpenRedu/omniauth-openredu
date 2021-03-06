# Omniauth Redu

Implementação de estratégia de autenticação do OAuth 2.0 para Ruby.

## Pré-requisitos

- [Ruby 1.8.7 ou superior](http://rvm.io/)
- [Bundler](http://gembundler.com/)
- [Aplicação Rails](http://guides.rubyonrails.org/)

## Instalação e uso

### Dependências

Adicione as seguintes dependências no seu Gemfile:

```ruby
gem 'omniauth-openredu', :git => 'https://github.com/Openredu/omniauth-openredu.git'
gem 'omniauth-oauth2', '~> 1.3.1' 
```

Execute o comando ``bundle install`` para baixar e instalar a dependência.

### Tokens

[Registre sua aplicação](https://openredu.ufpe.br/aplicativos/criar) no Redu e crie um initializer declarando o ``consumer_id`` e ``consumer_scret``:

```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :openredu,  "consumer_id", "consumer_secret"
end
```

### Rotas

É necessário fazer o mapeamente entre URL e actions nos controllers da sua aplicação. Para isso edite o arquivo ``config/routes.rb`` e adicione as seguintes rotas:

Neste exemplo é assumido que há um controller com o nome ``SessionsController`` que possui um action ``create``.

```ruby
  get '/auth/:provider/callback', :to => 'sessions#create'
  get '/auth/:provider', :to => 'sessions#create', as: :create_session
```

### Controller

Sempre que uma requisição for feita para a URL ``/auth/openredu`` é dado início ao ciclo de autorização. Ao fim do ciclo, o navegador do usuário é redirecionado para ``/auth/openredu/callback`` junto com o ``access_token`` do mesmo. O ``access_token`` deve ser persistido por sua aplicação e utilizado nas requisições subsequentes para pegar mais informações do usuário.

Um exemplo de ``SessionsController`` mais simples possível pode ser visto abaixo:

```ruby
class SessionsController < BaseController
  respond_to :html, :json

  def create
    # Busca ou cria usuário baseado no UID.
    if user = User.find_by_uid(auth_hash['uid']) || User.create_with_omniauth(auth_hash)

    # Informações para depuração
    Rails.logger.info "Adding #{user.id} to the session[:user_id]"

    # Adiciona ID do usuário à sessão
    session[:user_id] = user.id

    # Redireciona para URL da aplicação
    redirect_to root_path
  end

  protected

  def auth_hash
    request.env['omniauth.auth']
  end
end
```

O formado do hash disponível através de ``request.env['omniauth.openredu']`` é o seguinte:

```ruby
{
  "uid" => 1212, # ID do usuário no Openredu,
  "info" => {
    "name" => "Guilherme",
    "login" => "guiocavalcanti",
    "email" => "guilhermec@redu.com.br",
  },
  "credentials" => { "token" => "XXX" }
}
```

Uma imaplentação do método ``User.create_with_omniauth!`` pode ser vista abaixo:

```ruby
class User < ActiveRecord::Base

  def self.create_with_omniauth(auth)
    create! do |user|
      user.uid = auth["uid"]
      user.name = auth["info"]["name"]
      user.login = auth["info"]["login"]
      user.email = auth["info"]["email"]
      user.token = auth["credentials"]["token"]
    end
  end
end
```

## Mais exemplos de aplicatiovs

- Redu Canvas application to enable students to submit exercises: https://github.com/redu/submitit
- Aplicativo de cooperação anônima par a par para a plataforma social educacional Redu.: https://github.com/redu/autorregulacao

<img src="https://github.com/downloads/redu/redupy/redutech-marca.png" alt="Redu Educational Technologies" width="300">

This project is maintained and funded by [Redu Educational Techologies](http://tech.redu.com.br).

# Copyright

Copyright (c) 2012 Redu Educational Technologies

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.



