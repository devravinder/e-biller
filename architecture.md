### Naming Conventions
1. junctions should be always like  
   1. {groupA}_{groupB}_{direction}{number}  
      eg: clinets_server_h1  
      h = horizontal  
      v = vertical

   2. group_in or group_out  
   



```mermaid

architecture-beta

  junction server_data_v1
  junction server_data_v2
  junction server_data_v3
  junction server_data_v4


  junction data_consumers_v1
  junction data_consumers_v2
  junction data_consumers_v3
  junction data_consumers_v4

  junction data_consumers_h1

  junction data_micro_services_v1
  junction data_micro_services_v2
  junction data_micro_services_v3
  junction data_micro_services_v4
  junction data_micro_services_v5


  junction data_auth_service_v1
  junction data_auth_service_v2
  junction data_auth_service_v3
  junction data_auth_service_v4
  junction data_auth_service_v5

  junction data_auth_service_h1


  junction micro_services_v1
  junction micro_services_v2
  junction micro_services_v3

  junction micro_services_in_v1
  junction micro_services_in_v2
  junction micro_services_in_v3


  junction server_in_v1
  junction server_in_v2
  junction server_in_v3
  junction server_in_v4

  junction server_queue_v1
  junction server_queue_v2
  junction server_queue_v3

  junction clients_payment_v1

  junction auth_clients_h1
  junction auth_clients_h2
  junction auth_clients_h3


  junction auth_clients_v1
  junction auth_clients_v2
  junction auth_clients_v3

  

  group user_data(database)[User Data]
  service data_postgres(logos:postgresql)[Postgres] in user_data
  service data_redis(logos:redis)[ Redis ] in user_data 
  service data_elastic(logos:elasticsearch)[Elastic] in user_data
  service data_files(logos:aws-s3)[Files] in user_data


  server_data_v1:B --> T:server_data_v2
  server_data_v2:B --> T:server_data_v3
  server_data_v3:B <-- T:server_data_v4


  data_redis:L --> R:server_data_v1
  data_elastic:L --> R:server_data_v2
  data_postgres:L --> R:server_data_v3
  data_files:L --> R:server_data_v4


  data_consumers_v1:B -- T:data_consumers_v2
  data_consumers_v2:B -- T:data_consumers_v3
  data_consumers_v3:B -- T:data_consumers_v4

  data_redis:R --> L:data_consumers_v1
  data_elastic:R --> L:data_consumers_v2
  data_postgres:R --> L:data_consumers_v3
  data_files:R --> L:data_consumers_v4



  group main_service(server) [Main Server]
  service main_server(logos:aws-ec2)[ Java Server] in main_service
  service sse_notifications(logos:aws-sqs)[ Notifications ] in main_service 

  main_server:T --> B:sse_notifications

  main_server:R <-- L:server_data_v3


  group client_apps(internet) [Client Apps]
  service web_app(logos:react)[Web App] in client_apps
  service admin_web(logos:react)[Admin App] in client_apps


  server_in_v1:B --> T:server_in_v2
  server_in_v2:B <-- T:server_in_v3
  server_in_v3:B <-- T:server_in_v4
  
  admin_web:R --> L:server_in_v1
  web_app:R --> L:server_in_v2


  server_in_v2:R --> L:main_server

  group payment_gateway(server)[Payment Gateway]
  service payment_server(logos:aws-ec2)[ Node Server] in payment_gateway 
  service payment_postgres(logos:postgresql)[Postgres] in payment_gateway 

  payment_server:B<--T:payment_postgres
  payment_server:R --> L:server_in_v4

  clients_payment_v1:T <-- B:web_app
  clients_payment_v1:B --> T:payment_server

  group micro_services [Micro Services]

  group pdf_service(server)[Pdf Server] in micro_services
  service pdf_server(logos:nodejs-icon)[Node Server] in pdf_service

  group email_service(server)[Mail Server] in micro_services
  service email_server(logos:google-gmail)[Node Server] in email_service

  group cron_jobs(server)[Cron Jobs] in micro_services
  service cron_jobs_server(server)[Node Server] in cron_jobs


  micro_services_v1:B -- T:micro_services_v2 
  micro_services_v2:B -- T:micro_services_v3


  micro_services_in_v1:B -- T:micro_services_in_v2 
  micro_services_in_v2:B -- T:micro_services_in_v3



  pdf_server:R --> L:micro_services_v1
  email_server:R --> L:micro_services_v2
  cron_jobs_server:R --> L:micro_services_v3

  pdf_server:L <-- R:micro_services_in_v1
  email_server:L <-- R:micro_services_in_v2
  cron_jobs_server:L <-- R:micro_services_in_v3

  group message_queue(server)[Message Queue]
  service apache_kafka(logos:aws-sqs)[Apache Kafka] in message_queue
  
  apache_kafka:R --> L:micro_services_in_v2

  main_server:B --> T:server_queue_v1
  server_queue_v1:B --> T:server_queue_v2
  server_queue_v2:B --> T:server_queue_v3
  server_queue_v3:B --> T:apache_kafka


  data_micro_services_v1:B <-- T:data_micro_services_v2
  data_micro_services_v2:B <-- T: data_micro_services_v3
  data_micro_services_v3:B <-- T:data_micro_services_v4
  data_micro_services_v4:B <-- T:data_micro_services_v5


  micro_services_v2:R -- L:data_micro_services_v5

  data_consumers_v3:R --> L:data_micro_services_v1



  group auth_service(server)[Auth Service]
  service auth_server(logos:aws-iam)[Auth Server] in auth_service

  data_auth_service_v1:B <-- T:data_auth_service_v2
  data_auth_service_v2:B <-- T:data_auth_service_v3
  data_auth_service_v3:B <-- T:data_auth_service_v4
  data_auth_service_v4:B <-- T:data_auth_service_v5

  data_auth_service_v5:B <-- T:data_micro_services_v1

  data_auth_service_h1:R <-- L:data_auth_service_v1

  auth_server:R <-- L:data_auth_service_h1

  auth_clients_h1:R <-- L:auth_clients_h2
  auth_clients_h2:R <-- L:auth_clients_h3
  auth_clients_h3:R <-- L:auth_server

  auth_clients_v1:B --> T:auth_clients_v2
  auth_clients_v2:B --> T:auth_clients_v3
  auth_clients_v3:B --> T:server_in_v1

  auth_clients_h1:B --> T:auth_clients_v1






```