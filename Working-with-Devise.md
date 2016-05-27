### Registrations

app/controllers/registrations_controller.rb

```rb
class RegistrationsController < Devise::RegistrationsController
  def create
    build_resource(sign_up_params)
    if verify_rucaptcha?(resource) && resource.save
      yield resource if block_given?
      if resource.persisted?
        if resource.active_for_authentication?
          set_flash_message! :notice, :signed_up
          sign_up(resource_name, resource)
          respond_with resource, location: after_sign_up_path_for(resource)
        else
          set_flash_message! :notice, :"signed_up_but_#{resource.inactive_message}"
          expire_data_after_sign_in!
          respond_with resource, location: after_inactive_sign_up_path_for(resource)
        end
      else
        clean_up_passwords resource
        set_minimum_password_length
        respond_with resource
      end
    else
      clean_up_passwords resource
      respond_with resource
    end
  end
end
```

### Login

app/controller/sessions_controller.rb

```rb
class SessionsController < Devise::SessionsController
  prepend_before_action :valify_captcha!, only: [:create]

  def valify_captcha!
    unless verify_rucaptcha?
      redirect_to new_user_session_path, alert: 'Invalid captcha code'
      return
    end
    true
  end
end
```


### Password reset

app/controllers/passwords_controller.rb

```rb
class PasswordsController < Devise::PasswordsController
  def create
    self.resource = resource_class.find_or_initialize_with_errors(Devise.reset_password_keys, resource_params, :not_found)
    if self.resource.persisted? && verify_rucaptcha?(resource)
      self.resource.send_reset_password_instructions
    end

    yield resource if block_given?

    if successfully_sent?(resource)
      respond_with({}, location: after_sending_reset_password_instructions_path_for(resource_name))
    else
      respond_with(resource)
    end
  end
end
```

Now you can add RuCaptcha view code into Devise views.

And then change `routes.rb`:

```rb
devise_for :users, controllers: {
  registrations: :registrations,
  sessions: :sessions,
  passwords: :passwords
}
```