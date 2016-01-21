---
layout: post
title: "How to start active_admin with existing model"
date: 2014-04-03 11:20:20 +0800
comments: true
category: technical
tags: [gems, activeadmin]
---


There is a User model, and we want all admins are from users. So the big problem is How to config active_admin to support User model.

#### Step 1: Migrate table

{% highlight ruby%}
# ==> File: /db/migrate/xxxx_add_admin_to_users.rb

def change
  add_column :users, :admin, :boolean, :null => false, :default => false
end
{% endhighlight %}

#### Step 2: Reconfiguring ActiveAdmin

{% highlight ruby%}
# ==> File: config/initializers/active_admin.rb

config.authentication_method = :authenticate_admin!
config.current_user_method   = :current_user
config.logout_link_path      = :destroy_user_session_path
config.logout_link_method    = :delete

{% endhighlight %}

#### Step 3: Add Admin Authenticate Function

{% highlight ruby%}
# ==> File: app/controllers/application_controller.rb

def authenticate_admin!
  unless current_user.admin
    redirect_to root_path, :alert => "Unauthorized Access!"
  end
end

{% endhighlight %}

#### Step 4: Reconfiguring Routes

{% highlight ruby%}

# ==> File: config/routes.rb

ActiveAdmin.routes(self)

{% endhighlight %}

#### Step 5: Routes With Admin Namespace

{% highlight ruby%}

# ==> File: config/routes.rb

namespace :admin do
  constraints AdminConstraint do
    # mount Sidekiq::Web => '/sidekiq'
  end
end


# ===> config/initializers/admin_constraint.rb

class AdminConstraint
  def self.matches?(request)
    current_user = request.env['warden'].user
    current_user && current_user.admin?
  end
end

{% endhighlight %}
