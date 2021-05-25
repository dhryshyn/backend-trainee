## Практическая работа 0: Создание простейшего API с использованием NodeJS и express

## Начало работы

Задача:

Необходимо разработать API сервер для авторизации пользователей. Пользователи должны иметь возможность зарегистрироваться в системе указав емейл, юзернейм и пароль; авторизироваться при помощи логина и пароля; получить информацию своего профайла (для авторизированных пользователей). Авторизация должна производиться посредством [JWT токена](https://habr.com/ru/post/340146/).

Используемые технологии:

1. Node.js
2. Express
3. MongoDB

API дожно содержать следующие ендпоинт: `/user/signup` - для регистрации нового пользователя; `/user/auth` - для авторизации; `/user/profile` - для получения профайла пользователя;
Самый простейший флоу регистрации и авторизации можно описать так:
При регистрации пользователь в форме на клиенте вводит емейл, юзернейм и пароль. Эти данные передаются на сервер где валидируются. Если данные валидны, то создается новая запись в базе данных, формируется JWT токен в котором зашифровывается id новосозданного пользователя, затем токен отправляется на клиент, где сохраняется в local storage. 
При авторизации, пользователь в форме на клиенте вводит емейл и пароль. На сервере по емейлу находится нужный пользователь, после чего сличаются пароли (на самом деле сличаются хеши паролей). Если есть совпадение, вновь таки, формируется токен с зашифрованным в нем id пользователя, затем токен отправляется на клиент.

## Предварительные требования

Перед началом работы, нужно установить Node.js и npm на свой ПК. 
Node.js это окружение которое позволяет писать на js на сервере. Npm это пакетный менеджер который мы будем использовать чтобы устанавливать зависимости в наш проект. 
Переходим по [ссылке](https://nodejs.org/en/) и устанавливаем актуальную  версию Node.js. 
Чтобы убедиться, что все установилось корректно в консоли выполняем команду 
`node -v`, а затем `npm -v`.
Если оба компонента установлены корректно, то в консоли выведется версия. 

Окей, теперь у нас есть нода, и мы можем выполнять js код на сервере, но помимо этого нам нужно где-то хранить данные. Для этого будем использовать базу mongoDB. Чтобы облегчить установку, создадим базу в облаке. 
Регистрируемся на сайте [mongoDB](https://www.mongodb.com/try) для создания базы в облаке и следуем этому [туториалу](https://www.youtube.com/watch?v=rPqRyYJmx2g) (Интерфейс в видео может слегка отличаться). 
Когда мы все сделали, у нас будет создана база и добавлен пользователь для этой базы. Далее мы должны сгенерировать код для подключения к базе и скопировать его себе. Как это сделать тоже показывается в даном туториале  [туториале](https://www.youtube.com/watch?v=rPqRyYJmx2g).

Сгенерированный код для подключения будет выглядеть примерно так 

`mongodb+srv://<username>:<password>@cluster0.paxev.mongodb.net/myFirstDatabase?retryWrites=true&w=majority`

Мы создали базу, установили ноду, но перед тем как начать писать код нам надо создать папку проекта и установить туда все зависимости. 
Создаем папку проекта и устанавливаем в нее необходимые модули. Для этого пишем в консоли

`mkdir myFirstApp`
`cd myFirstApp`
`npm install express express-validator body-parser bcryptjs jsonwebtoken mongoose --save`

После того как команда выполнится, в проекте появится папка node_modules и package.json. Это регистр наших зависимостей. Теперь пару слов о зависимостях. 

Express это фреймворк для создания API, express-validator - библиотека для валидации данных, body-parser - библиотека для парсинга body в http запросе который будет приходить на наш сервер bcrypt - библиотека которая содержит алгоритмы шифрования (для шифрования пароля), jsonwebtoken - библиотека для создания и валидации токена.

## Создаем заготовку чтобы проверить работоспособность проекта
После всех предыдущих шагов у нас есть все чтобы начать писать код. Внутри папки проекта создаем файл index.js и делаем минимальную заготовку. 

```
const express = require("express");
const bodyParser = require("body-parser");

const app = express();

// PORT
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
  res.json({ message: "API Working" });
});


app.listen(PORT, (req, res) => {
  console.log(`Server Started at PORT ${PORT}`);
});
```
Как видно в коде, мы подключили express и создали один рутовый роут, затем сказали express-у на каком порту развернуть сервер. 
Сохраняем код и запускаем команду `node index.js`. 
Если видим в консоли текст `Server Started at 3000`, значит сервер “поднялся”. Теперь мы можем тестировать наши API endpointы локально. 
Открываем браузер, пробуем зайти на адрес [http://localhost:3000/](http://localhost:3000/)
В отладчике смотрим, что нам в ответ пришло { message: "API Working" }. 
Значит все работает. 

Далее тестируем подключение к базе данных. Создаем файл db.js

```const mongoose = require("mongoose");

const MONGOURI = "сюда вставить строку которую мы получили на 1 шаге";

const InitiateMongoServer = async () => {
  try {
    await mongoose.connect(MONGOURI, {
      useNewUrlParser: true
    });
    console.log("Connected to DB !!");
  } catch (e) {
    console.log(e);
    throw e;
  }
};

InitiateMongoServer(); 
```
Сохраняем код и запускает с помощью команды `node db.js`.
Если видим в консоли сообщение об успешном подключении, переходим к следующим шагам. Если видим ошибку подключения, то возвращаемся на шаг 1)  и проверяем все ли мы сделали правильно, смотрим еще раз туториал. Если не помогло, спрашиваем в чат =)

## Создаем модель пользователя

Когда у вас получилось успешно запустить API сервер и подключиться к базе, создадим модель пользователя в файле models/user.js
```
//FILENAME : models/user.js
const mongoose = require("mongoose");

const UserSchema = mongoose.Schema({
  username: {
    type: String,
    required: true
  },
  email: {
    type: String,
    required: true
  },
  password: {
    type: String,
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now()
  }
});

// export model user with UserSchema
module.exports = mongoose.model("user", UserSchema);
```
Таким образом, мы объявили какие поля будут присутствовать у пользователя, и какого типа эти поля.

Теперь слегка модифицируем наш файл db.js, и добавим возможность импортировать функцию ```InitiateMongoServer``` и уберем вызов этой функции файле db.js.

```
//FILENAME : db.js

const mongoose = require("mongoose");

const MONGOURI = "сюда вставить строку которую мы получили на 1 шаге";

const InitiateMongoServer = async () => {
  try {
    await mongoose.connect(MONGOURI, {
      useNewUrlParser: true
    });
    console.log("Connected to DB !!");
  } catch (e) {
    console.log(e);
    throw e;
  }
};

module.exports = InitiateMongoServer;

```

 Соберем все воедино  в файле index.js:
 ```
const express = require("express");
const bodyParser = require("body-parser");
const InitiateMongoServer = require("./db");

// Initiate Mongo Server
InitiateMongoServer();

const app = express();

// PORT
const PORT = process.env.PORT || 3000;

// Middleware
app.use(bodyParser.json());

app.get("/", (req, res) => {
  res.json({ message: "API Working" });
});


app.listen(PORT, (req, res) => {
  console.log(`Server Started at PORT ${PORT}`);
});
```
Обратите внимание что мы добавили миддлвар bodyParser и теперь мы можем работать с http body.
