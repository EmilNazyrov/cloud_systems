# Цель
Изучить так называемые "плохие практики" (антирекомендации) по написанию docker-файлов и научиться их исправлять, чтобы получить верный грамотно составленный файл.

# Ход работы

## Плохой docker-файл

Ниже представлен код плохого docker-файла:

```dockerfile
FROM python:3

RUN apt-get update
RUN apt-get install -y python3-pip
RUN pip install flask

COPY . /app

WORKDIR /app
CMD ["python3", "app.py"]

Чем он плох?

 1. Используется слишком общий базовый образ без указания версии. В таком случае Docker выберет новейшую версию Python 3.x из доступных, что по сути эквивалентно использованию тега latest.
 2. Три конструкции RUN.
 3. Нет очистки кэша.
 4. Запуск от root.
 5. Копирование всех файлов из текущей директории.

Хороший docker-файл

Далее приведём код хорошего docker-файла:

FROM python:3.9.7

RUN apt-get update && apt-get install -y --no-install-recommends python3-pip && \
    pip install --no-cache-dir flask && \
    rm -rf /var/lib/apt/lists/*

COPY requirements.txt /app/
RUN pip install --no-cache-dir -r /app/requirements.txt
COPY . /app

RUN addgroup --system appgroup && adduser --system --group appuser
USER appuser

WORKDIR /app
CMD ["python3", "app.py"]

Что исправлено?

 1. Указан базовый образ с версией 3.9.7 (конкретная версия).
 2. Только одна конструкция RUN вместо трёх.
 3. Есть очистка кэша.
 4. Запуск не от root.
 5. Указаны конкретные файлы для копирования.

Плохие практики при работе с контейнерами

 1. Неправильное управление ресурсами
Контейнеры могут потреблять больше ресурсов, чем им нужно, если не задавать ограничения. Это приводит к снижению производительности других приложений.
Пример:

docker run myapp

Исправленный вариант:

docker run --memory="512m" --cpus="1.0" myapp


 2. Запуск контейнеров с повышенными привилегиями
Если запустить контейнер с повышенными привилегиями, это может привести к нарушению изоляции. Этот контейнер сможет получить доступ к критическим ресурсам.
Пример:

docker run --privileged myapp

Исправленный вариант:

docker run --cap-add=NET_ADMIN myapp
