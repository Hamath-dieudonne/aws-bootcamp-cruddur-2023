FROM python:3.10-slim-buster

# Inside container
# Make a new folder inside container
WORKDIR /backend-flask

# Outside container -> Inside container
# this contains the libraries we want to install to run the app
COPY requirements.txt requirements.txt

# Inside container
# Install the python libraries used for the app
RUN pip3 install -r requirements.txt

# means everything in the current directory
# first period - /backend-flask ( Inside container)
# secon period - /backend-flask ( Outside container)
COPY . .

# Set environment variables (Env vars)
# Inside container and will remain set when the container is running
ENV FLASK_ENV=development

EXPOSE ${PORT}

# CMD (Command)
# python3 -m flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]