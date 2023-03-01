# Week 1 — App Containerization

## Create Dockerfile in `backend-flask`

Create `Dockerfile` using this code:

```python
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
````

 • open port `4567`

## Run Python

```python
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
```
 • click the link next to `4567`
 
 • add `/api/activities/home`at the end of the url to display json:
 
 ```json [
  {
    "created_at": "2023-02-24T00:04:09.277108+00:00",
    "expires_at": "2023-03-03T00:04:09.277108+00:00",
    "handle": "Andrew Brown",
    "likes_count": 5,
    "message": "Cloud is fun!",
    "replies": [
      {
        "created_at": "2023-02-24T00:04:09.277108+00:00",
        "handle": "Worf",
        "likes_count": 0,
        "message": "This post has no honor!",
        "replies_count": 0,
        "reply_to_activity_uuid": "68f126b0-1ceb-4a33-88be-d90fa7109eee",
        "reposts_count": 0,
        "uuid": "26e12864-1c26-5c3a-9658-97a10f8fea67"
      }
    ],
    "replies_count": 1,
    "reposts_count": 0,
    "uuid": "68f126b0-1ceb-4a33-88be-d90fa7109eee"
  },
  {
    "created_at": "2023-02-19T00:04:09.277108+00:00",
    "expires_at": "2023-03-07T00:04:09.277108+00:00",
    "handle": "Worf",
    "likes": 0,
    "message": "I am out of prune juice",
    "replies": [],
    "uuid": "66e12864-8c26-4c3a-9658-95a10f8fea67"
  },
  {
    "created_at": "2023-02-25T23:04:09.277108+00:00",
    "expires_at": "2023-02-26T12:04:09.277108+00:00",
    "handle": "Garek",
    "likes": 0,
    "message": "My dear doctor, I am just simple tailor",
    "replies": [],
    "uuid": "248959df-3079-4947-b847-9e0892d1bab4"
  }
]
````

## Build a Container:

```bash
docker build -t  backend-flask ./backend-flask
```

## Run the Container:

```python
docker run --rm -p 4567:4567 -it backend-flask
FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"
````

## Run Container in the background:

```docker container run --rm -p 4567:4567 -d backend-flask
````

## Run `npm install`:

```bash
cd frontend-react-js
npm i
```
## Create Dockerfile in `frontend-react-js/Dockerfile`:

```python
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```
## Build Container:

`docker build -t frontend-react-js ./frontend-react-js`

## Run the Container:

`docker run -p 3000:3000 -d frontend-react-js`

## Create a `docker-compose.yml`file at the root of the project:

```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
 ```
 
## Troubleshooting

Left my laptop open and Gitpod shut down. When I logged back in and started everything up, the text was missing from the Cruddur app page:


![Text Missing](assets/TextMissing(1).png)

Tried `compose restart`

![Compose Restart](assets/ComposeRestart.png)

Then I checked to see if the ports were on and open:

![NotFound](assets/4567NotFound.png) ![Found 4567 Not Running](assets/Found4567NotRunning.png)

Then I ran `docker ps`to get the Container ID but because it was not running there was no ID so then I ran `docker ps -a` which shows all Containers regardless if they are on or not and used that to run `docker logs` to see what issues there were.

```python
Traceback (most recent call last):
  File "/usr/local/lib/python3.10/runpy.py", line 196, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "/usr/local/lib/python3.10/runpy.py", line 86, in _run_code
    exec(code, run_globals)
  File "/usr/local/lib/python3.10/site-packages/flask/main.py", line 3, in <module>
    main()
  File "/usr/local/lib/python3.10/site-packages/flask/cli.py", line 1050, in main
    cli.main()
  File "/usr/local/lib/python3.10/site-packages/click/core.py", line 1055, in main
    rv = self.invoke(ctx)
  File "/usr/local/lib/python3.10/site-packages/click/core.py", line 1657, in invoke
    return _process_result(sub_ctx.command.invoke(sub_ctx))
  File "/usr/local/lib/python3.10/site-packages/click/core.py", line 1404, in invoke
    return ctx.invoke(self.callback, ctx.params)
  File "/usr/local/lib/python3.10/site-packages/click/core.py", line 760, in invoke
    return __callback(*args, kwargs)
  File "/usr/local/lib/python3.10/site-packages/click/decorators.py", line 84, in new_func
    return ctx.invoke(f, obj, args, **kwargs)
  File "/usr/local/lib/python3.10/site-packages/click/core.py", line 760, in invoke
    return __callback(args, *kwargs)
  File "/usr/local/lib/python3.10/site-packages/flask/cli.py", line 911, in run_command
    raise e from None
  File "/usr/local/lib/python3.10/site-packages/flask/cli.py", line 897, in run_command
    app = info.load_app()
  File "/usr/local/lib/python3.10/site-packages/flask/cli.py", line 312, in load_app
    app = locate_app(import_name, None, raise_if_not_found=False)
  File "/usr/local/lib/python3.10/site-packages/flask/cli.py", line 218, in locate_app
    import(module_name)
  File "/backend-flask/app.py", line 7, in <module>
    from services.notifications_activities import
  File "/backend-flask/services/notifications_activities.py", line 5
    results = [{
IndentationError: unexpected indent
```

This indicated that I had an identation error in my code, which I found in line 5 in my `notifications_activities.py`file and I corrected that and it worked! :)

Then later on my whole Cruddur App page dissappeared :(

![Page Dissappeared](assets/PageDissappeared.png)

So I did all the above steps and also hit inspect. I tried to reboot and when I ran the `npm i` and it was returning some errors:


```npm ERR! code ENOENT```

Checked on Google and found an informative blog post which said:

![npmi](assets/npmi.png)

Link to the full article: https://sebhastian.com/npm-err-enoent

I found that I had deleted the packet.json file so I copied the code from omenking repo and created a new file and it worked again!

## Add Postgres to `docker-compose.yml`:

```yaml
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

## Add DynamoDB to `docker-compose.yml`file:

```yaml
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```

## Add a new Endpoint for notifications in `open api`:

 • add a new path in `open api`:
 
 ```python
 /api/activities/notifications:
    get:
      description: 'Return a feed of activity for all of those that I follow'
      tags: 
        - activities
      parameters: []
      responses:
        '200':
          description: Returns an array of activities
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Activity' 
```
## Add a Backend Endpoint:

In `app.py`file add:

`from services.notifications_activities import *` 

Then add:

```python
@app.route("/api/activities/notifications", methods=['GET'])
def data_notifications():
  data = NotificationsActivities.run()
  return data, 200
  ```
  
• add a `notifications.py`file in `backend-flask/services``

```python
from datetime import datetime, timedelta, timezone
class NotificationsActivities:
  def run():
    now = datetime.now(timezone.utc).astimezone()
    results = [{
      'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
      'handle':  'Andrew Brown',
      'message': 'Cloud is very fun!',
      'created_at': (now - timedelta(days=2)).isoformat(),
      'expires_at': (now + timedelta(days=5)).isoformat(),
      'likes_count': 5,
      'replies_count': 1,
      'reposts_count': 0,
      'replies': [{
        'uuid': '26e12864-1c26-5c3a-9658-97a10f8fea67',
        'reply_to_activity_uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
        'handle':  'Worf',
        'message': 'This post has no honor!',
        'likes_count': 0,
        'replies_count': 0,
        'reposts_count': 0,
        'created_at': (now - timedelta(days=2)).isoformat()
      }],
    }
    ]
    return results
 ```
 
 








