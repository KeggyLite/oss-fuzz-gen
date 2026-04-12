> **Форк проекта [google/oss-fuzz-gen](https://github.com/google/oss-fuzz-gen)**

# Автоматический фаззинг cJSON с помощью OSS-Fuzz-Gen

#### 1. развертывание  
#### 2. конект с cJSON (коммит 3a7bd69)
#### 3. Зафиксировать ограничения 


### 1. Развертывание
Для развертывания в домашней среде была выбранна ОС Ubuntu 22.04 на гипервизоре VB.
После старотовой установки было установленно следующее окружение в соответствии с USAGE.md:
- Python 3.1
- pip
- venv
- Git
- Docker
- API-KEY OpenRouter
- 
#### 1.1 Простые компоненты

##### Установка Python 3.11+pip:
```bash
sudo apt update
sudo apt install -y python3.11 python3.11-dev python3.11-venv
sudo apt install -y python3-pip
```
##### Установка venv
```bash
sudo apt install -y python3.11-venv
```
##### Установка Git
```bash
sudo apt update
sudo apt install -y git
```
##### Установка Docker
```bash
# 1. Обновление пакетов и установка зависимостей
sudo apt update
sudo apt install -y ca-certificates curl

# 2. Добавление официального ключа Docker
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 3. Добавление репозитория Docker
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. Установка Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 5. Добавление пользователя в группу docker (чтобы не использовать sudo)
sudo usermod -aG docker $USER

# 6. Применение изменений (требуется перезапуск сессии или выполнить команду ниже)
newgrp docker
```
##### Проверка установленных комплектующих
```bash
# Проверка Python 3.11
python3.11 --version

# Проверка pip
pip3 --version

# Проверка venv
python3.11 -c "import venv; print('venv: OK')"

# Проверка Git
git --version

# Проверка Docker
docker --version
```
##### Ожидаемый результат:
<img width="669" height="461" alt="изображение" src="https://github.com/user-attachments/assets/6d0664b7-c5a3-43b2-a627-f1402792fed6" />

#### 1.2 API-key
в данном эксперементе испоользовался ключ предоставляемый OpenRouter:
Регистрация на OpenRouter -> Генерация API-ключа-> сохранить кей(предоставляется единожды)
![Без имени](https://github.com/user-attachments/assets/b2aa5ae4-6516-4a52-ab34-641165690a27)

![Без имени](https://github.com/user-attachments/assets/1a7fbf55-9571-46dc-96fc-58032e9fb5aa)

далее необходимо передать API-ключ в ВМ через переменные окружения
```bash
export OPENROUTER_API_KEY="sk-or-v1-33dd***___****89f2d2f"
export OPENAI_API_KEY="$OPENROUTER_API_KEY"
export OPENAI_API_BASE="https://openrouter.ai/api/v1"
```
проверка, что ключ работает корректно на примере модели в бесплатном доступе 
```bash
 curl -X POST https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openai/gpt-oss-120b:free",
    "messages": [{"role": "user", "content": "Say OK in one word"}],
    "max_tokens": 10
  }'
```
ожидаемый результат в консоли + трек на сайте OpenRout
![Без имени](https://github.com/user-attachments/assets/eff81112-314a-48ff-8549-8074cf48a751)
<img width="669" height="587" alt="изображение" src="https://github.com/user-attachments/assets/b714f906-01e7-48e0-9767-4764e716493a" />

### 2 был проведен ряд тестов для запуска механизма аналогичных приведенному в USAGE.md получавшие те или иные res в терминале:

ряд ошибок констатировался выбором неподдерживаемой модели.

```bash
./run_all_experiments.py \
    -y ./benchmark-sets/all/tinyxml2.yaml \
    --model openai/gpt-oss-120b \
    --work-dir ./results_gptoss \
    --max-trials 2
```

```log
**** FINAL RESULTS: ****


2026-04-11 04:34:00.615 INFO run_all_experiments - _print_experiment_results: ================================================================================
*tinyxml2, void tinyxml2::XMLElement::SetAttribute(const char *, const char *)*
Exception while running experiment: Bad model type openai/gpt-oss-120b

2026-04-11 04:34:00.615 INFO run_all_experiments - _print_experiment_results: ================================================================================
*tinyxml2, XMLNode * tinyxml2::XMLElement::ShallowClone(XMLDocument *)*
Exception while running experiment: Bad model type openai/gpt-oss-120b

2026-04-11 04:34:00.615 INFO run_all_experiments - _print_experiment_results: ================================================================================
*tinyxml2, XMLAttribute * tinyxml2::XMLElement::FindOrCreateAttribute(const char *)*
Exception while running experiment: Bad model type openai/gpt-oss-120b

2026-04-11 04:34:00.615 INFO run_all_experiments - _print_experiment_results: ================================================================================
*tinyxml2, char * tinyxml2::XMLElement::ParseDeep(char *, StrPair *, int *)*
Exception while running experiment: Bad model type openai/gpt-oss-120b

2026-04-11 04:34:00.615 INFO run_all_experiments - _print_experiment_results: ================================================================================
*tinyxml2, bool tinyxml2::XMLPrinter::VisitEnter(const XMLElement &, const XMLAttribute *)*
Exception while running experiment: Bad model type openai/gpt-oss-120b

2026-04-11 04:34:00.615 INFO run_all_experiments - _print_experiment_results: **** TOTAL COVERAGE GAIN: ****
```

на гпт 3.5 с рядом корректив вывод стал следующим:
```bash
./run_all_experiments.py \
    -y ./benchmark-sets/all/tinyxml2.yaml \
    --model gpt-3.5-turbo \
    --work-dir ./results_openrouter
```
```log
2026-04-11 04:39:28.534 INFO _client - _send_single_request: HTTP Request: POST https://api.openai.com/v1/chat/completions "HTTP/1.1 403 Forbidden"
2026-04-11 04:39:28.534 WARNING models - with_retry_on_error: LLM API Error when responding (attempt 3): Error code: 403 - {'error': {'code': 'unsupported_country_region_territory', 'message': 'Country, region, or territory not supported', 'param': None, 'type': 'request_forbidden'}}
2026-04-11 04:39:28.535 WARNING models - _delay_for_retry: Retry in 41 seconds...
2026-04-11 04:39:31.555 INFO _client - _send_single_request: HTTP Request: POST https://api.openai.com/v1/chat/completions "HTTP/1.1 403 Forbidden"
2026-04-11 04:39:31.556 WARNING models - with_retry_on_error: LLM API Error when responding (attempt 3): Error code: 403 - {'error': {'code': 'unsupported_country_region_territory', 'message': 'Country, region, or territory not supported', 'param': None, 'type': 'request_forbidden'}}
2026-04-11 04:39:31.556 WARNING models - _delay_for_retry: Retry in 45 seconds..
```





