FROM python:3.10-slim
ENV APP_MODE=production
ENV APP_REGION=canada-west
WORKDIR /app
COPY . . 
RUN pip install --no-cache-dir flask
CMD ["python","app.py"]
EXPOSE 8080
