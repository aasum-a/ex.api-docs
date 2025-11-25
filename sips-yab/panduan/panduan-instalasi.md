# Getting Started 

Follow these steps to get a local copy of the project up and running.

### 1. Clone the Repository

First, clone this repository to your local machine.

#### Ganti dengan URL repositori Anda
```bash
git clone [https://github.com/your-username/your-repository.git](https://github.com/your-username/your-repository.git)
cd your-repository
```


### 2. Install Dependencies
Install all the required dependencies for both PHP and Node.js.

#### Install PHP dependencies
```bash
composer install
```

#### Install Node.js dependencies
```bash
npm install
```

### 3 Environment Configuration

This is the most crucial step. You will configure your local environment variables.

#### a. Create `.env` file

Copy the example environment file. All your local configurations will be stored here.

```bash
cp .env.example .env
```

#### b. Generate Application Key

Every Laravel application needs a unique application key.

```bash
php artisan key:generate
```

#### c. Configure Database

Open the `.env` file and update the database connection details to match your local setup.

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE={Your Database Name}
DB_USERNAME=root
DB_PASSWORD=
```

#### d. Generate JWT Secret Key

This project uses JWT for authentication. Generate a secret key by running the following command:

```bash
php artisan jwt:secret
```

> **Note:** If a secret key already exists and you need to regenerate it, use the `--force` flag:
> `php artisan jwt:secret --force`

#### e. Configure Captcha

The project uses Captcha Provider to protect forms from spam. Configure the Captcha Provider in the `.env` file.

```ini
CAPTCHA_DRIVER="{Your Captcha Driver}"
CAPTCHA_SITE_KEY="{Your Site Key}"
CAPTCHA_SECRET_KEY="{Your Site Secret}"
CAPTCHA_LOCALE="id"
```
For more details, visit the [official documentation](https://laravel-captcha.rahuldey.dev/docs/2.x/).

#### f. Configure WhatsApp Bot

This application integrates with a WhatsApp bot for sending notifications. You need to run the [wa-bot-yab](https://github.com/IT-YAB/wa-bot-yab) service separately.

1.  Clone and run the bot service following its repository instructions.
2.  Once the bot is running, update the `.env` file with its URL and API key.

<!-- end list -->

```ini
APP_WHATSAPP_URL={Your Bot URL} #URL of your running bot, for example: http://127.0.0.1:5000/api/broadcast
APP_WHATSAPP_KEY="{Your Secret Key}" #The secret key from the bot's configuration
APP_WHATSAPP_OTP_TIMEOUT=2 #in minute
```

### 4 Database Migration

Run the database migrations to create all the necessary tables. The `--seed` flag will also populate the database with initial data.

```bash
php artisan migrate:fresh && php artisan db:seed
```

### 5 Create Storage Link

To make your uploaded files in `storage/app/public` accessible from the web, create a symbolic link.

```bash
php artisan storage:link
```

> **Troubleshooting:** If the command fails because `public/storage` already exists, delete the directory first and then run the command again or use Manual Symlink Creation (Advanced).

### 6 Add Required Boilerplate File

This project requires a specific boilerplate file to function correctly.

1.  **Download the file** from the following link:
      - **Link:** [Download Boilerplate File from Google Drive](https://drive.google.com/file/d/19nWT8-uONGYI61-SHm_bXVX21ktsebxV/view?usp=sharing)
2.  **Place the file** into the `storage/app/` folder of your project.

-----

## Running the Development Server 🖥️

This project uses a single command to start both the PHP server and the Vite asset bundler.

**To start the server, run:**

```bash
composer run dev
```

  - Your application will be available at **`http://127.0.0.1:8000`**.
  - The Vite server will handle hot-reloading for your CSS and JavaScript assets.

To stop both servers, press `Ctrl + C` in the terminal.

> ✨**Tip:** If u want run on another port then change the port number in `vite.config.js` for vite and `.env` for php.
-----

## Available Scripts

Here are some of the most common scripts available for this project.

| Command             | Description                                       |
| :------------------ | :------------------------------------------------ |
| `composer run dev`  | Starts both the Vite and PHP development servers. |
| `npm run dev`       | Starts only the Vite development server for assets. |
| `npm run build`     | Compiles and minifies assets for production.      |
| `php artisan serve` | Starts only the Laravel PHP development server.   |
| `php artisan test`  | Runs the application's test suite (PHPUnit).      |
| `php artisan migrate` | Runs the database migrations.                     |

### Documentations for Google Drive Storage
for more update and more doc you can hit from this repo `https://github.com/yaza-putu/laravel-google-drive-storage`
