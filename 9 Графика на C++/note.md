# Приложение 1. Графика на C++

```cpp
#include <X11/Xlib.h>

#include <stdlib.h>
#include <unistd.h>

#include <iostream>

int main()
{
  Display *d; // Дисплей -место, где отображается результат (монитор и т.д.)
  // Window   w; // Экран - рабочий стол
  int      s;
  
  d = XOpenDisplay(nullptr);
  if ( d == nullptr )
  {
  exit(1);
  }
  
  s = DefaultScreen(d); // Получить информацию об экране по умолчанию
  
  // Метод ниже открывает окно. Параметры:
  // 1. Дисплей
  // 2. Главный экран
  // 3. X окна
  // 4. Y окна
  // 5. Ширина окна
  // 6. Высота окна
  // 7. Толщина рамки
  // 8. Цвет рамки
  // 9. Фон окна
  w = XCreateSimpleWindow( d, RootWindow(d,s), 
  10,  10,
  200, 100,
  1,
  BlackPixel(d,s), WhitePixel(d,s));
  
  XMapWindow(d,w); // Отобразить окно
  
  XFlush(d);
  
  XSelectInput(d,w,
  ButtonPressMask | ButtonReleaseMask |
  KeyPressMask | KeyReleaseMask |
  ExposureMask );
  
  while(1)
  {
  XEvent event; // Создаём объект события
  
  XNextEvent(d,&event); // Выдёргиваем событие из очереди
  
  switch(event.type) // Выбираем событие из очереди
  {
    case Expose: // Отображение окна
    XClearWindow(d,w); // Очистка старых рисунков в окне
    
    // Ниже приведён метод печати строки
    // 1. Дисплей
    // 2. Окно
    // 3. Графический контекст по умолчанию
    // 4. X строки
    // 5. Y строки
    // 6. Текст
    // 7. Длина строки
    XDrawString(d,w,
      DefaultGC(d,s),
      20,30,
      "Hello, World!", 13); 
    
    break;
  
    case ButtonPress: // +Release : mouse
    break;
    case KeyPress: // +Release : keyboard
    break;
  }
  }
  
  XCloseDisplay(d); // Закрыть соединение с дисплеем
  
  return 0;
}
```
