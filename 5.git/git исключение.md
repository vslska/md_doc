
```bash
# Добавь правило в локальный .git/info/exclude (не попадёт в коммит!)
echo "hypr/img/*" >> .git/info/exclude
echo "!hypr/img/logo-catppuccin.png" >> .git/info/exclude
```

В exclude добавить каталог, но применить оператор "! "   (оператор НЕ )