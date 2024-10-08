# lanzar_instancia_RoR
instrucciones para lanzar una instancia de ruby on rails
# realestate-nft-marketplace

- Crear el servidor en Lightsail
- Settear una ip estatica para el servidor
- Actualizar los puertos del server: settear https y 3000
- Actualizar los registros del dominio
- Instalar Nginx, configurar Cerbot y Passenger
- https://www.phusionpassenger.com/docs/tutorials/deploy_to_production/deploying_your_app/oss/aws/ruby/nginx/
  - Acceder al servidor por ssh
  - sudo apt update
  - sudo apt-get install nginx
  - sudo apt-get install libnginx-mod-http-headers-more-filter
  - https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal
  - sudo snap install core; sudo snap refresh core
  - sudo apt-get remove certbot
  - sudo snap install --classic certbot
  - sudo ln -s /snap/bin/certbot /usr/bin/certbot
  - https://www.phusionpassenger.com/docs/advanced_guides/install_and_upgrade/nginx/install/oss/focal.html
  - sudo apt-get install -y dirmngr gnupg apt-transport-https ca-certificates
  - sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
  - sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger focal main > /etc/apt/sources.list.d/passenger.list'
  - sudo apt-get update
  - sudo apt-get install -y libnginx-mod-http-passenger
  - if [ ! -f /etc/nginx/modules-enabled/50-mod-http-passenger.conf ]; then sudo ln -s /usr/share/nginx/modules-available/mod-http-passenger.load /etc/nginx/modules-enabled/50-mod-http-passenger.conf ; fi
  - sudo ls /etc/nginx/conf.d/mod-http-passenger.conf
    - El archivo /etc/nginx/conf.d/mod-http-passenger.conf debe existir
  - sudo service nginx restart
  - Validar configuracion con:
    - sudo /usr/bin/passenger-config validate-install
- Crear Nginx Virtual Host
  - Acceder al servidor por ssh
  - cd /etc/nginx/sites-available
  - sudo nano landagreenstore.com.conf
  - configurar este archivo
    -encontrar el path de ruby utiliza el comando
    -passenger-config about ruby-command 
    -referencia:
           server {
                listen 80;
                server_name landagreenstore.com www.landagreenstore.com 52.38.48.164;
            
                # Redireccionar todo el tráfico HTTP a HTTPS
                return 301 https://$host$request_uri;
            }
            
            server {
                listen 443 ssl; # Gestión SSL
                server_name landagreenstore.com www.landagreenstore.com;
            
                # Configuración del root
                root /var/www/landagreenstore.com/public;
            
                # Configuración de Passenger
                passenger_enabled on;
                passenger_app_env production;
                passenger_ruby /usr/share/rvm/gems/ruby-3.2.2/wrappers/ruby;
            
            
                # Manejo de las peticiones
                location / {
                    try_files $uri/index.html $uri.html $uri @app;
                }
            
                location @app {
                    passenger_enabled on;
                    passenger_app_env production; # Asegúrate de que esto esté presente
                }
            
                # Manejo de errores
                error_page 500 502 503 504 /500.html;
                location = /500.html {
                    root /var/www/landagreenstore.com/public;
                }
            
                # Configuración de SSL
                ssl_certificate /etc/letsencrypt/live/www.landagreenstore.com/fullchain.pem; # Gestionado por Certbot
                ssl_certificate_key /etc/letsencrypt/live/www.landagreenstore.com/privkey.pem; # Gestionado por Certbot
                include /etc/letsencrypt/options-ssl-nginx.conf; # Gestionado por Certbot
                ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # Gestionado por Certbot
            }

- Instalar backend
  - Instalar rvm
    - https://github.com/rvm/ubuntu_rvm
    - sudo apt-get install software-properties-common
    - sudo apt-add-repository -y ppa:rael-gc/rvm
    - sudo apt-get update
    - sudo apt-get install rvm
    - sudo usermod -a -G rvm $USER
    - echo 'source "/etc/profile.d/rvm.sh"' >> ~/.bashrc
    - Reiniciar
    - rvm list
    - rvm list known
    - rvm install 2.7.2
    - rvm use 2.7.2
  - Instalar postgres dependencies
    - sudo apt-get -y install libpq-dev
  - Instalar codigo
    - cd /home/ubuntu/.ssh/
    - ssh-keygen
    - Registrar ssh key en github
      - cat id_rsa.pub
      - pegar en github ssh keys
    - cd /var/
    - sudo chown ubuntu www -R
    - clonar el repo en www
    - git clone git@github.com:UVG-Teams/realestate-nft-marketplace.git
    - mv realestate-nft-marketplace/ vesta.f-rosal.com/   
    - cd vesta.f-rosal.com
    - BUNDLER_WITHOUT="development:test" bundle install
    - git checkout production
  - Instalar DB
    - Crear DB en Lightsail
    - Settear credenciales
      - EDITOR="nano" rails credentials:edit --environment production
      -  *crea un strin con rake secret y guardalo en este archivo como:
          secret_key_base: "unstringrandomquesetrocurraogeneresenalgunlado"
        - Settear db credentials
        - Settear secret_key_base
    - RAILS_ENV=production rails db:drop
    - RAILS_ENV=production rails db:create
    - RAILS_ENV=production rails db:migrate
    - RAILS_ENV=production rails db:migrate:status
    - RAILS_ENV=production rails db:seed
    - Guardar passwords
  - Crear symbolic links para habilitar el sitio (virtual host setteado previamente)
    - sudo ln -s /etc/nginx/sites-available/vesta.f-rosal.com.conf /etc/nginx/sites-enabled/
    - cd /etc/nginx/sites-enabled
    - sudo nginx -t
    - sudo service nginx restart
    - sudo service nginx reload
    - sudo service nginx force-reload
    - sudo service nginx restart
  - Configurar certbot
    - sudo certbot --nginx
    - Confirmar que funciona visitando https
  - sudo service nginx restart
  - cd /var/www/vesta.f-rosal.com/
  - add writing permissions to public folder
    - sudo chmod 755 public/ -R
  - make current logged user as owner of public folder
    - sudo chown ubuntu public/ -R
  - make nginx user as owner of public folder
    - sudo chown www-data:www-data public/ -R
  - sudo service nginx restart
- Intalar Frontend
  - On local
    - npm run build
  - Upload build/
  - https://github.com/Olafaloofian/React-Frontend-Lightsail-Deployment
  - sudo curl -sL https://deb.nodesource.com/setup_10.x -o nodesource_setup.sh
  - sudo bash nodesource_setup.sh
  - sudo apt-get install nodejs
  - sudo apt-get install build-essential
  - cd /var/www/
  - mkdir spa.vesta.f-rosal.com
  - sudo ln -s /var/www/vesta.f-rosal.com/lib/spa/build/* /var/www/spa.vesta.f-rosal.com
  - sudo service nginx restart
  - https://create-react-app.dev/docs/deployment
  - sudo npm install -g serve
  - serve -s build -l 80
  - sudo service nginx restart
