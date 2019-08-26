# Send Emails

Send emails with nodemailer and put in redis queue

<p align="center">*</p>

# Nodemailer

```sh
yarn add nodemailer
```

<p align="center">*</p>

**src/config/mail.js**

```js
export default {
  host: '',
  port: '',
  secure: false,
  auth: {
    user: '',
    pass: '',
  },
  default: {
    from: 'Equipe <noreply@bobarber.com>'
  },
}
```

<p align="center">*</p>

**src/lib/Mail.js**

```js
import nodemailer from 'nodemailer'
import mailConfig from  '../config/mail'

class Mail {
  constructor() {
    const { host, port, secure, auth } = mailConfig;

    this.transporter = nodemailer.createTransport({
      host,
      port,
      secure,
      auth: auth.user ? auth : null,
    })
  }

  sendMail(message) {
    return this.transporter.sendMail({
      ...mailConfig.default,
      ...message
    })
  }
}

export default new Mail();
```

<p align="center">*</p>

# Templates

```sh
yarn add express-handlebars nodemailer-express-handlebars 
```

<p align="center">*</p>

**src/lib/Mail.js**

```js
//...
import { resolve } from 'path'
import exphbs from 'express-handlebars'
import nodemailerhbs from 'nodemailer-express-handlebars'

//...

configureTemplates() {
  const viewPath = resolve(__dirname, '..', 'app', 'views', 'emails')

  this.transporter.use('compile', nodemailerhbs({
    viewEngine: exphbs.create({
      layoutsDir: resolve(viewPath, 'layouts'),
      partialsDir: resolve(viewPath, 'partials'),
      defaultLayout: 'default',
      extName: '.hbs'
    })
  }))
}
```

<p align="center">*</p>

**src/app/views/emails/layouts/default.hbs**

```hbs
<div style="font-family: Arial, Helvetica, sans-serif; font-size: 16px; line-height: 1.6; color: #222; max-width: 600px; margin:0 auto; text-align: center">
  {{{ body }}}
  {{**footer }}
</div>

```

<p align="center">*</p>

**src/app/views/emails/partials/default.hbs**

```hbs
<br/>
<hr/>
<em>Equipe GoBarber</em>
```

<p align="center">*</p>

**src/app/views/emails/cancellation.hbs**

```hbs
<strong>Olá, {{provider}}</strong><br/>
<p>
  Houve um cancelamento de horário. Confira detalhes abaixo:
</p>
<p>
  <strong>Cliente: </strong**{{user}} <br/>
  <strong>Data/hora: </strong**{{date}}<br/>
  <br/>
  <small>O horário está disponível para novos agendamentos.</small>
</p>

```

<p align="center">*</p>

# Redis

```sh
docker run --name newredisname -p 6379:6379 -d -t redis:alpine
```

<p align="center">*</p>

**Add bee queue**

```sh
yarn add bee-queue
```

<p align="center">*</p>

**src/config/redis**

```js
export default {
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
};
```

<p align="center">*</p>

With the file bellow we can start queue without main app

**src/queue.js**
```js
import 'dotenv/config';
import Queue from './lib/Queue';

Queue.processQueue();

```

<p align="center">*</p>

**src/lib/Queue.js**

```js
import Bee from 'bee-queue';
import redisConfig from '../config/redis';
import CancellationMail from '../app/jobs/CancellationMail';

const jobs = [CancellationMail];

class Queue {
  constructor() {
    this.queues = {};
    this.init();
  }

  init() {
    jobs.forEach(({ key, handle }) =**{
      this.queues[key] = {
        bee: new Bee({
          redis: redisConfig,
        }),
        handle,
      };
    });
  }

  add(queue, job) {
    return this.queues[queue].bee.createJob(job).save();
  }

  processQueue() {
    jobs.forEach(job => {
      const { bee, handle } = this.queues[job.key];

      bee.on('failed', this.handleFailure).process(handle);
    });
  }

  handleFailure(job, err) {
    console.log(`Queue ${job.queue.name} FAILED: ${err}`);
  }
}

export default new Queue();

```

<p align="center">*</p>

**src/app/jobs/CancellationMail.js**

```js
import { format, parseISO } from 'date-fns';
import pt from 'date-fns/locale/pt';
import Mail from '../../lib/Mail';

class CancellationMail {
  get key() {
    return 'CancellationMail';
  }

  async handle({ data }) {
    const { appointment } = data;

    await Mail.sendMail({
      to: `${appointment.provider.name} <${appointment.provider.email}>`,
      subject: 'Agendamento Cancelado',
      template: 'cancellation',
      context: {
        provider: appointment.provider.name,
        user: appointment.user.name,
        date: format(parseISO(appointment.date), "dd 'de' MMMM', às' H:mm'h'", {
          locale: pt,
        }),
      },
    });
  }
}

export default new CancellationMail();
```

<p align="center">*</p>

**src/app/controllers/AppointmentController.js**

```js
import Queue from '../../lib/Queue.js'


//...async delete()

  await Queue.add(CancellationMail.key, { appointment });

//...

```