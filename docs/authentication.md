# Example for Authenticating a User

in routes.rb
```rb
Rails.application.routes.draw do
  # ...... your other routes
  unless Rails.env.production?
    scope path: "/__cypress__", controller: 'cypress' do
      post "forceLogin", action: 'force_login'
    end
  end
end
```

app/controllers/cypress_controller.rb
```rb
class CypressController < ApplicationController
  skip_before_action :verify_authenticity_token

  def force_login
    if params[:email].present?
      user = User.find_by!(email: params.require(:email))
    else
      user = User.first!
    end
    sign_in(user)
    redirect_to URI.parse(params.require(:redirect_to)).path
  end
end
```

in cypress/support/on-rails.js
```js
Cypress.Commands.add("forceLogin", (details) => {
  if(!details)
    details = {}

  if(!details.redirect_to)
    details.redirect_to = '/'

  cy.visit("__cypress__/forceLogin",
    { method: "POST", body: { email: details.email, redirect_to: details.path }  })
});
```

examples of usage in cypress specs
```js
cy.forceLogin()
cy.forceLogin({redirect_to: '/profile'})
cy.forceLogin({email: 'someuser@mail.com'})
```
