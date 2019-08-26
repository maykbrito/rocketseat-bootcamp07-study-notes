# Notifications

Notification System


$$*$$

**Install local MongoDB**

```sh
docker run --name nameofyournewmongodb -p 27017:27017 -d -t mongo
```

$$*$$

**connect to db**

```sh
yarn add mongoose
```

$$*$$

**add mongo connection**
```js
import mongoose from 'mongoose'

mongoose.connect(
  'mongodb://localhost:27017/nameofyournewdatabase',
  { useNewUrlParser: true, useFindAndModify: true }
)
```

$$*$$

To see logs:  `docker logs`

 $$*$$   

# Starting

**src/app/schemas/Notification.js**

```js
import mongoose from 'mongoose';

const NotificationSchema = new mongoose.Schema({
  content: {
    type: String,
    required: true,
  },
  user: {
    type: Number,
    required: true,

  },
  read: {
    type: Boolean,
    required: true,
    default:false,
  },
  timestamps:true,
})

export default mongoose.model('Notification', NotificationSchema)
```

$$*$$

# Create a Notification

**src/app/controllers/AppointmentController.js**

```js
import Notification from 'NotificationSchema'
import { format } from 'date-fns'
import pt from 'date-fns/locale/pt'
//...


const user = await User.findByPk(req.userId)
const formattedDate = format(
  hourStart,
  "dd 'de' MMMM', às' H:mm'h'",
  { locale: pt }
)

/**
 *  Notify appointment provider
 */

await Notification.create({
  content: `Novo agendamento de ${user.name} para dia ${formattedDate}`,
  user: provider_id,
});

//...

```

$$*$$

# List and mark as read

**src/routes.js**

```js
//...

routes.get('/notifications', NotificationController.index);
routes.put('/notifications:id', NotificationController.update);

//...

```

$$*$$

**src/app/controllers/NotificationController.js**

```js

import User from '../models/User'
import Notification from  '../schemas/Notification'

class NotificationController {

  async index(req, res) {
    const checkIsProvider = await User.findOne({
      where: { id: req.userId, provider: true },
    })

    if (!checkIsProvider)
      return res.status(401).json({ error: 'Sorry. Providers only' })
    
    const notifications = await Notification.find({
      user: req.userId,
    }).sort({ createdAt: 'desc' })
      .limit(20)

    return res.json(notifications)
  }

  async update(req, res) {
    const notification = await Notification.findByIdAndUpdate(
      req.params.id,
      { read: true },
      { new: true}
    )

    return res.json(notification)
  }
}

export default new NotificationController();

```