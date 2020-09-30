# Getting Started on Heroku with PHP
---
## Heroku dengan PHP

1.	Check kebutuhan
 
<div align="center"><img src="gambar/php/version_php.png" width="500px"></div>

<div align="center"><img src="gambar/php/version_composer.png" width="500px"></div>

<div align="center"><img src="gambar/php/version_git.png" width="300px"></div>
 
2.	Login Heroku 

<div align="center"><img src="gambar/php/login.png" width="500px"></div>

3.	Mempersiapkan APP

<div align="center"><img src="gambar/php/clone.png" width="500px"></div>
 
4.	Deploy App

<div align="center"><img src="gambar/php/create.png" width="500px"></div>

<div align="center"><img src="gambar/php/push.png" width="500px"></div>
 
5.	Menjalankan aplikasi

<div align="center"><img src="gambar/php/ps.png" width="500px"></div>

<div align="center"><img src="gambar/php/php.png" width="500px"></div>
 
6.	Melihat logs

<div align="center"><img src="gambar/php/log1.png" width="800px"></div>
 
 <div align="center"><img src="gambar/php/log2.png" width="800px"></div>

## Mendifine procfile

1.	Check berapa banyak dynos yang running

<div align="center"><img src="gambar/php/ps2.png" width="500px"></div>

2.	Scaling aplikasi di Heroku 

<div align="center"><img src="gambar/php/scale0.png" width="500px"></div>


## Declare dependensi dari suatu aplikasi

1.	Install dependensi

<div align="center"><img src="composer_update.png" width="800px"></div>


## Melakukan perubahan

1.	Menambahkan dependensi di composer

<div align="center"><img src="gambar/php/composer_cowsay.png" width="500px"></div>
 
2.	Update composer

<div align="center"><img src="gambar/php/composer_update2.png" width="500px"></div>

3.	Lakukan perubahan di file index.php

<div align="center"><img src="gambar/php/index.png" width="500px"></div>
 
4.	Tambahkan perubahan ke git repository

<div align="center"><img src="gambar/php/gitadd.png" width="500px"></div> 

<div align="center"><img src="gambar/php/push.png" width="500px"></div>
 
5.	Buka app di web yang sudah dilakukan perubahan: ```open cowsay```

<div align="center"><img src="gambar/php/cowsay2.png" width="500px"></div>
 

## Penyediaan add-on

**NOTE : Untuk add-on diharuskan menggunakan account Heroku yang telah terverifikasi dengan credit card.**


## Memulai dengan tampilan shell
1.	Menjalankan di console, check environment

<div align="center"><img src="gambar/php/consol.png" width="500px"></div>

2.	Check file yang ada di dyno

<div align="center"><img src="gambar/php/bash.png" width="500px"></div>


## Define config vars
1.	Modifikasi file index.php

<div align="center"><img src="gambar/php/hello.png" width="500px"></div>
 
2.	Setting dan check config var di Heroku

<div align="center"><img src="gambar/php/config.png" width="500px"></div>

<br>
<br>

***Sumber : [Getting Started on Heroku with PHP](https://devcenter.heroku.com/articles/getting-started-with-php?singlepage=true)***