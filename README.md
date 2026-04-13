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

![Без имени](https://github.com/user-attachments/assets/b85b668e-9754-4a7b-b342-0120fbcd0f48)

<img width="613" height="108" alt="взаимосвязь" src="https://github.com/user-attachments/assets/57f65f76-5377-4526-8d85-312452e73660" />


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

добавление cJSON 

<img width="815" height="1166" alt="2222" src="https://github.com/user-attachments/assets/b0a160ec-201d-4ab5-b2fd-528eafeabd78" />

### 2 был проведен ряд тестов, для запуска механизма, аналогичных приведенному в USAGE.md получавшие те или иные res:


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
-----
на гпт 3.5 с рядом корректив вывод стал следующим:

<img width="609" height="463" alt="Снимок экрана от 2026-04-13 22-58-24" src="https://github.com/user-attachments/assets/60bfe5dd-9071-4caa-a5cb-ca7e0b45125a" />
<img width="609" height="873" alt="Снимок экрана от 2026-04-13 23-00-46" src="https://github.com/user-attachments/assets/a6289e81-ee2b-4b8f-9b14-1172960f1100" />

----
<img width="611" height="585" alt="Снимок экрана от 2026-04-13 23-12-21" src="https://github.com/user-attachments/assets/3754b487-aef0-452d-b0d6-05a1e397a4ae" />

<img width="611" height="1027" alt="Снимок экрана от 2026-04-13 23-10-24" src="https://github.com/user-attachments/assets/d71e30a6-4793-4484-9bab-e68b0323a5cd" />



далее подогнав параметры под представленные в USAGE.md получен следующий рес

<img width="755" height="199" alt="изображение" src="https://github.com/user-attachments/assets/e6a9fc92-7617-4158-9d89-25a27cb25960" />

на данном этапе было выясненно что oss-fuzz работает только с образами предпрописанными и определенными в models.py в папке llm_toolkit.
была осуществлена попытка подмены класса в моделях

<img width="707" height="493" alt="11111" src="https://github.com/user-attachments/assets/66f9de75-f6ac-4fef-8362-5c4c729e1991" />

но в результате программа зациклилась выводя пустые шаблоны папок

<img width="761" height="943" alt="Снимок экрана от 2026-04-14 00-05-22" src="https://github.com/user-attachments/assets/a0188c9d-f2f8-4a60-9564-743c2546b36b" />

так же при всех тестах направленных на cJSON и иные софтины трек OpenRouter не обновлялся, что несложно складывается в то что OpenRouter апишник не разу не был принят фазером
![Без имени](https://github.com/user-attachments/assets/1b6f9e7a-74e3-45f4-9359-3ee1dbd61d71)

### 3 эндинг

"из коробки" и около данного фразеологизма использовать опенфаз оказалось достаточно проблематичным

имеет место быть ограничение на LLM (за пределами OpenAI и Google Cloud SDK), как обойти и возможно ли это сделать вообще (кроме как выйти за пределы предложенного в models.py)
