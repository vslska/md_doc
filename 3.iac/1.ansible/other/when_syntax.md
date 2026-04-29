
# When - syntx

### 1. База

Внутри `when` **запрещено** использовать двойные фигурные скобки `{{ }}`. Все выражение по умолчанию считается Jinja2-шаблоном.

- **Не норм:** `when: "{{ my_var }} == 'ready'"`
- **Норм:** `when: my_var == 'ready'`

---
### 2. Базовые проверки

| Условие                       | Описание                                              |
| ----------------------------- | ----------------------------------------------------- |
| `when: var_name is defined`   | Переменная существует                                 |
| `when: var_name is undefined` | Переменная НЕ существует                              |
| `when: var_name`              | Переменная равна `true` (или не пустая строка/список) |
| `when: not var_name`          | Переменная равна `false` (или пуста)                  |
| `when: var_name == "value"`   | Строгое сравнение строк                               |

---
### 3. Логические операторы и списки

Для объединения условий используйте `and`, `or` или YAML-списки (которые работают как `and`).

**Вариант со списком (рекомендуется для AND):**

```yaml
when:
  - ansible_os_family == "Debian"
  - ansible_distribution_version is version('22.04', '>=')
  - my_feature_enabled | bool
```

**Вариант с OR:**
```yaml
when: ansible_distribution == "CentOS" or ansible_distribution == "RHEL"
```

---
### 4. Версионность и пакеты

Для проверки версий ОС или ПО используйте тест `version`.

```yaml
when: ansible_distribution_version is version('24.04', '>=')
```

---
### 5. Проверки на основе выполнения (Register)

Используйте `register`, чтобы сохранить результат задачи и проверить его позже.
```
- name: Проверка конфига
  command: /usr/bin/verify_config
  register: config_check
  ignore_errors: true

- name: Перезапуск сервиса
  service: name=myapp state=restarted
  when: config_check.rc == 0 and config_check.changed
```

>_Новинка 2026:_ В экспериментальных сборках (core 2.21+) появляется **Register Projections**, позволяющий извлекать конкретные данные прямо в момент регистрации через Jinja-выражения.

---
### 6. Работа с файлами и путями

Для проверки наличия файлов на **удаленном** узле используйте модуль `stat`.

```yaml
- stat: path=/etc/app/config.yml
  register: cfg

- debug: msg="Файл существует"
  when: cfg.stat.exists
```

---
### 7. Современные тесты и фильтры (Best Practices 2026)

- **Безопасность:** Всегда используйте `| bool` для переменных, которые могут прийти как строки ("yes", "true", "1").
    - `when: my_var | bool`
- **Undefined:** Используйте тест `none`, если переменная может быть объявлена, но иметь пустое значение.
    - `when: my_var is not none`
- **Списки:** Проверка наличия элемента в списке.
    - `when: "'web' in group_names"`

---
### 8. Отладка условий

Если `when` не работает как ожидалось, используйте `ansible-lint`. Он автоматически подсветит лишние скобки или некорректные пробелы в Jinja2-выражениях.

**Как проверить текущие факты для условий:**

```bash
ansible hostname -m setup | less
```