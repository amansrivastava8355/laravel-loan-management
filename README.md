<h1 align="center">Laravel Loan Management System API</h1>

## Setting up Local Development Environment

It is recommended to use Docker with Laravel Sail for a quick and reliable setup.

- Create a new Docker Network: ```docker network create --driver bridge my-network```
- Pull mySQL image: ```docker pull mysql/mysql-server```
- Run mySQL Container: ```docker run --network my-network --name mysql -d -p 3308:3306 -v ~/mysql:/var/lib/mysql -e MYSQL_ROOT_HOST=% -e MYSQL_ROOT_PASSWORD=root mysql/mysql-server --default_authentication_plugin=mysql_native_password```
- Create Redis Container: ```docker run --network my-network --name redis -d -p 6380:6379 redis```
- Create Mailhog Container: ```docker run --network my-network --name mailhog -d -p 1025:1025 -p 8025:8025 mailhog/mailhog```

Clone this repo: ```git clone https://github.com/amansrivastava8355/laravel-loan-management```


Install Composer:
```
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v $(pwd):/opt \
    -w /opt \
    laravelsail/php81-composer:latest \
    composer install --ignore-platform-reqs
```

Build App Docker Container:
```
./vendor/bin/sail build
```

Set following keys in .env:

```
APP_PORT=8000
APP_SERVICE=app
COMPOSE_PROJECT_NAME=COMPOSE_PROJECT_NAME=lambdatest_loan_management
```

Fire up Laravel Sail:

```
./vendor/bin/sail up -d
```
Note: Create `sail` alias: https://laravel.com/docs/8.x/sail#configuring-a-bash-alias



# About

This is a API first approach application built to manage a Loan Management System.
These API's can be consumed by any front end tech stack be it in react, angular or offcourse any mobile app.
Currently the Application is built over Laravel Framework version 9, and it was last tested on laravel version 9.29.0

It is an app that allows authenticated users to go through a loan application. It doesn’t have to contain too many fields, but at least “amount
required” and “loan term.” All the loans will be assumed to have a “weekly” repayment frequency.
After the loan is approved, the user must be able to submit the weekly loan repayments. It can be a simplified repay functionality, which won’t need to check if the dates are correct but will just set the weekly amount to be repaid.

## Choices I made for the application

    - used Spatie Roles and Permissions package - https://spatie.be/docs/laravel-permission/v5/introduction
        - https://laravel-news.com/two-best-roles-permissions-packages
        - It has a very clear documentation and easy to understand
        - we can use core laravel's gate methods like `can` to check on any permissions created using this package

    - Created a `LoanService` to separate out the business logic in the service and allow controller do what its job is,
        - accept the request from the route
        - send it to the loan service to process
        - get back the response from the service
        - finally send a json reponse back to the user

    - Created Separate Request classes for request validation
    - created `LoanPolicy` to manage different permissions while access different routes

## Github repo

https://github.com/amansrivastava8355/laravel-loan-management

## Postman API Document

https://documenter.getpostman.com/view/3532076/2s93kz5QBe

## Features

-   Customer creates a loan
-   Admin approves the loan
-   Customer can only view self owned loan
-   Customer can repay the loan only once Admin approves the loan
-   Once customer pays all the scheduled payment the Loan is marked automatically marked as Paid

1. Customer create a loan:
   Customer submit a loan request defining amount and term example:
    - Request amount of 10.000 $ with term 3 on date 7th Feb 2022
    - customer will generate 3 scheduled repayments:
    - 14th Feb 2022 with amount 3.333,33 $
    - 21st Feb 2022 with amount 3.333,33 $
    - 28th Feb 2022 with amount 3.333,34 $
    - the loan and scheduled repayments will have state PENDING
2. Admin approve the loan:
    - Admin change the pending loans to state APPROVED
3. Customer can view loan belong to him:
    - Add a policy check to make sure that the customers can view them own loan only.
4. Customer add a repayments:
    - Customer add a repayment with amount greater or equal to the scheduled repayment
    - The scheduled repayment change the status to PAID
    - If all the scheduled repayments connected to a loan are PAID automatically also the loan become PAID

## Packages used

1. Spatie Roles & Permission Package - https://github.com/spatie/laravel-permission
    - for managing users with different permissions and give them access to the required / allowed services only

2. rename .env.example to .env file
3. run `./vendor/bin/sail php artisan migrate`
4. run `./vendor/bin/sail php artisan db:seed`
5. run `./vendor/bin/sail php artisan serve`
6. You can access the app at http://127.0.0.1:8000/

### All the API urls will of format

-   `/api/v1/<controller/method>`
-   If you were to access the app at http://127.0.0.1:8000/ then the login route API url will be
    -   `http://127.0.0.1:8000/api/v1/auth/login`

### List of avaiable End Points

I had a virtual host setup on my mac configured with `http://laravalloan.test` domain, so the routes looked like :

-   Auth Routes

    -   Register - `http://laravalloan.test/api/v1/auth/register`
    -   Login - `http://laravalloan.test/api/v1/auth/login`
    -   Logout - `http://laravalloan.test/api/v1/auth/logout`

-   Loan Routes
    -   Create Loan - `http://laravalloan.test/api/v1/loan/create`
    -   List Loans - `http://laravalloan.test/api/v1/loan/list`
    -   View Loan - `http://laravalloan.test/api/v1/loan/<loan_uuid>`
    -   Approve Loan - `http://laravalloan.test/api/v1/loan/approve`
    -   Repay Loan - `http://laravalloan.test/api/v1/loan/payment`

### Run Tests

-   Run all Tests at once
    -   `php artisan test`

-   Run individual Tests with below command

    -   Unit Test

        -   `php artisan test --filter=test_loan_service_methods`

    -   Feature Tests
        -   Auth Tests
            -   `php artisan test --filter=test_a_user_can_register`
            -   `php artisan test --filter=test_a_user_can_login`
        -   Loan Tests
            -   `php artisan test --filter=test_a_customer_can_create_loan`
            -   `php artisan test --filter=test_an_admin_can_approve_loan`
            -   `php artisan test --filter=test_a_customer_can_view_loan_list_of_only_self_with_no_loan_created`
            -   `php artisan test --filter=test_a_customer_can_view_loan_list_of_only_self`
            -   `php artisan test --filter=test_an_admin_can_view_loan_list_of_all_members`
            -   `php artisan test --filter=test_a_customer_can_view_only_self_loan`
            -   `php artisan test --filter=test_a_customer_cannot_view_other_users_loan`
            -   `php artisan test --filter=test_a_customer_can_repay_loan`
            -   `php artisan test --filter=test_a_customer_can_only_repay_a_loan_if_the_loan_belongs_to_him`
            -   `php artisan test --filter=test_a_loan_is_marked_as_paid_once_all_emis_are_paid`
