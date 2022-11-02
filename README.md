# battle-ship

Небольшой проект с моей версией интерактивной игры в морской бой, в который раньше играли на бумаге).  
Очень много логики, расчетов, активная работа с DOM-деревом, условные стили, небольшой UI-kit, декомпозиция компонентов, динамический подсчет ходов и обсчет повреждений, ведущаяся статистика раундов, доступно два вида читов). Компьютеру привита реалистическая моель поведения, то есть при попадании в корабль, он будет отстреливать смежные клетки до тех пор, пока не выведет его из строя. Компьютер стремится понять как ориентирован корабль а затем прицельно бьёт без промахов. Так что генерация точки для выстрела через Math.random() только при поиске цели). Написано на Vue 3.


## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


### Some screenshots
![Screenshot_1](https://user-images.githubusercontent.com/49340195/199443959-74446620-e300-458f-8ad0-ab14d6178115.png)
![Screenshot_2](https://user-images.githubusercontent.com/49340195/199444924-1841c9c9-925a-48a6-8d46-4484afaf44bb.png)
![Screenshot_3](https://user-images.githubusercontent.com/49340195/199444967-0dc4de64-40a7-417d-b206-32b204c7a60a.png)
![Screenshot_4](https://user-images.githubusercontent.com/49340195/199445009-84873ca6-9f85-4290-b799-53160a5c99c7.png)
![Screenshot_5](https://user-images.githubusercontent.com/49340195/199445034-b8da22f8-f272-4d13-aa91-a88821bd4c44.png)
![Screenshot_6](https://user-images.githubusercontent.com/49340195/199445056-19168cdf-fccd-4e10-ae1d-592c85de7157.png)
