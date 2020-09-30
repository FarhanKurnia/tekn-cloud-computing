# Getting Started on Heroku with Python
---
## Persiapan

1.	[Signup ke Heroku](https://signup.heroku.com/).

<div align="center"><img src="gambar/python/Signup_Heroku.png" width="500px"></div>

    Lalu setelah itu login

<div align="center"><img src="gambar/python/Login_Heroku.png" width="500px"></div>

2.	Buat aplikasi baru melalui dashboard

<div align="center"><img src="gambar/python/CreateNewApp_Heroku.png" width="500px"></div>

<div align="center"><img src="gambar/python/CreateNewApp_Heroku2.png" width="500px"></div>

3.	Install Python

<div align="center"><img src="gambar/python/Python_Version.png" width="300px"></div>


## Getting Started on Heroku with Python

1.	[Download Heroku CLI](https://cli-assets.heroku.com/heroku-x64.exe).

2.	Install Heroku cli

<div align="center"><img src="gambar/python/Heroku_version.png" width="300px"></div>

3.	Mempersiapkan APP

<div align="center"><img src="gambar/python/Heroku_Clone.png" width="500px"></div>

4.	Deploy APP

<div align="center"><img src="gambar/python/Heroku_Clone2.png" width="500px"></div>

<div align="center"><img src="gambar/python/Heroku_Clone3.png" width="500px"></div>

5.	Deploy code yang ada

<div align="center"><img src="gambar/python/Heroku_Deploy.png" width="500px"></div>

6.	Aplikasi telah terdeploy, pastikan sudah running

<div align="center"><img src="gambar/python/Heroku_Running.png" width="500px"></div>

7.	Buka APP yang sudah dibuat

<div align="center"><img src="gambar/python/Heroku_LocalWeb2.png" width="500px"></div>

8.	Melihat logs

<div align="center"><img src="gambar/python/Heroku_Log.png" width="800px"></div>

<div align="center"><img src="gambar/python/Heroku_Log2.png" width="800px"></div>


### Mendifine procfile

1.	Install Python dependencies

<div align="center"><img src="gambar/python/Heroku_ps.png" width="500px"></div>

2.	Melihat list dependenies

<div align="center"><img src="gambar/python/Heroku_ps_scale.png" width="500px"></div>


### Declare dependensi dari suatu aplikasi

1.	Check dynous yang running

<div align="center"><img src="gambar/python/Heroku_Requirement2.png" width="500px"></div>

2.	Scaling aplikasi di Heroku dengan mengganti nomor dynos yang running

<div align="center"><img src="gambar/python/Heroku_List.png" width="500px"></div>


### Menjalankan aplikasi secara local

1.	Menjalankan collectstatic untuk menjalankan secara local

<div align="center"><img src="img/py-12.png" width="500px"></div>

2.	Star APP secara local

<div align="center"><img src="img/py-13.png" width="500px"></div>

3.	Buka dengan [http://localhost:5000/](http://localhost:5000/)

<div align="center"><img src="img/py-06_db.png" width="500px"></div>


### Melakukan perubahan

1.	Melakukan perubahan dengan menambahkan dependencies

<div align="center"><img src="img/py-14.png" width="500px"></div>

<div align="center"><img src="img/py-15.png" width="300px"></div>

2.	Melakukan perubahan di file views.py

<div align="center"><img src="img/py-16.png" width="300px"></div>

3.	Buka Kembali [http://localhost:5000/] (http://localhost:5000/) setelah melakukan perubahan

<div align="center"><img src="img/py-17_db.png" width="200px"></div>

4.	Menyimpan perubahan ke repository di git

<div align="center"><img src="img/py-18.png" width="500px"></div>

<div align="center"><img src="img/py-19.png" width="500px"></div>
<div align="center"><img src="img/py-20.png" width="500px"></div>

5.	Cek di web aplikasi yang sudah dibuat

<div align="center"><img src="img/py-17_db.png" width="200px"></div>


### Penyediaan add-on

**NOTE : Untuk add-on diharuskan menggunakan account Heroku yang telah terverifikasi dengan credit card.**


### Menjalankan di console
1.	Menjalankan di console

<div align="center"><img src="img/py-21.png" width="500px"></div>

2.	Check file yang ada di dyno

<div align="center"><img src="img/py-22.png" width="500px"></div>


### Define config vars
1.	Edit views.py

<div align="center"><img src="img/py-23.png" width="300px"></div>
 
2.	Buka Kembali [http://localhost:5000/](http://localhost:5000/) setelah melakukan perubahan

<div align="center"><img src="img/py-24.png" width="200px"></div>

3.	Setting config var di Heroku 
 
<div align="center"><img src="img/py-25.png" width="500px"></div>

<br>
<br>

***Sumber : [Getting Started on Heroku with Python](https://devcenter.heroku.com/articles/getting-started-with-python?singlepage=true)***