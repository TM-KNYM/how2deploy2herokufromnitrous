## Nitrous.io(lite)でHerokuへDjangoをDepoly
無料のWebIDEであるnitrous.io liteを使って、ブラウザだけでWebアプリ（Django）を作成してみよう  
ついでに、herokuにデプロイしてみよう
### ゴール
http://XXXX.herokuapp.com をブラウザで開くと  
「Hello Nitrous & Heroku」と表示されるとこまで。  
XXXXにはアプリ名が入ります。
###注意
この説明ではDBやStaticファイルの設定方法は扱いません。  
あくまでもHerokuへデプロイすることが主目標です。

###事前に必要なもの

 - ブラウザ

以上   
WebIDEっていいね。


## 環境構築
環境構築とは名ばかりで、今回はアカウントを作成するだけです。
いずれも公式ページのSign up for freeからアカウントを作成すればOK
 1. Heroku　(https://www.heroku.com/)
 2. nitrous.io lite　(https://lite.nitrous.io/)

## NitrousでDjangoアプリを作成

早速、nitrous.ioでWebアプリ(django)を作ってみましょう
#### boxの作成
まず、Djnagoアプリ用のboxを作ります。boxとは実行環境のようなものです。
Python/Djangoでboxを作ると予め必要なものがインストールされた環境が使えるようになります。
1. nitrousにログインしてnew boxを選択
2. 言語の選択とアプリ名を決めます。言語は色々選べますが、今回はPython/djnagoを選択。アプリ名は適当でOKです。
3. create boxをクリックして完成
4. IDEを選択すると、その名の通りIDEが起動します。
最後に確認のために、IDE下部にあるconsole画面にpython -Vと打つとpythonのバージョン情報を表示  

#### Webアプリ作成
ここを参考にしています。ただ、DBまわりは飛ばします。
http://help.nitrous.io/django-app/
nitrous.ioはvituralenv推奨(仮想環境)なので、vituralenvを入れます。

    $pip install virtualenv
現状のディレクトリ構成は下記のようになっているので、workspace下に仮想環境とプロジェクトを作成します。
**ディレクトリ構成**

    ~/
		workspace/
		README.md
    
**手順**

    $cd workspace
    $mkdir venv 
    $virtualenv venv
    $cd venv  
    $source bin/activate
    
左端に(venv)と出ればOK  
ようやくここでdjangoをインストールします

     $pip install django-toolbelt

ここでポイントとしてはdjango-toolbeltでインストールすること  
Herokuに必要なパッケージも入るのでオススメです  
さくっとdjangoのプロジェクトを作ります  
今回のプロジェクト名はdeploy_testとします  

    $django-admin.py startproject deploy_test
    

そして、サーバーを立ち上げてみる

    $cd deploy_test
    $python manage.py runserver 0.0.0.0:8888

プレビューで確認してみましょう  
上部メニューから下記のように辿ると  
**preview**  >  **port8888**  
ブラウザが開いてIt works!と出ればOKです。  
サーバを停止させるときはctrl+cで止まります。  
次にgunicornでサーバーが起動出来るか確認してみます。  
gunicornとはHerokuで推奨されているWSGIサーバーです。  

     $gunicorn deploy_test.wsgi -b 0.0.0.0:8888

再度、**preview**  >  **port8888**で動作確認。  
先ほどと同じようにIt works!と出ればひとまずOKです。  

**現状のディレクトリ構成**

    ~/workspace
    /venv
        /bin
        /deploy_test <=カレントはここ
            /deploy_test
        /include
        /lib
        /local

つぎに、ItsWorks!ではなく「Hello Nitrous&Heroku」と表示させるようにします。  
まず、viewとなるviews.pyをvenv/deploy_test/deploy_testに作成

    $cd ~/workspace/venv/deploy_test/deploy_test
    $touch views.py

views.pyを以下のように編集して保存

    from django.http import HttpResponse
    def index(request):
	    return HttpResponse("Hello Nitrous & Heroku")

次に同じ階層にあるurls.pyを以下のように修正

    from django.conf.urls import include, url
    from . import views
    urlpatterns = [
	    url(r'^$', views.index, name='index'),
	    #url(r'^admin/', include(admin.site.urls)),
	    #adminは今回は使わないのでコメントアウト
    ]
サーバーを起動。

    $gunicorn deploy_test.wsgi -b 0.0.0.0:8888

プレビューからブラウザを開いて  
「Hello Nitrous & Heroku」  
とブラウザに表示されれば今回作るアプリは完成です。

### Herokuへデプロイ
いよいよデプロイします  
まず、djangoのプロジェクトディレクトリ下に移動します。  
ここが作業するディレクトリになります。  

    $cd ~/workspace/venv/deploy_test/

#### 下準備
下準備としてherokuにアップロードするのに必要なファイルを作ります。
その１ requirements.txtの作成 
requirements.txtとは、Herokuに対してPython必要なパッケージを教えるために必要です。

    $pip freeze > requirements.txt
    #中身は以下のようになっているはず。
    dj-database-url==0.3.0
    dj-static==0.0.6
    Django==1.8.2
    django-toolbelt==0.0.1
    gunicorn==19.3.0
    psycopg2==2.6.1
    static3==0.6.1

その２ Procfileの作成  
ProcfileとはHerokuがサーバーを起動する時に実行するコマンドを定義するものです。

    $touch Procfile

Procfileの中身は以下ようにします。

    web: gunicorn deploy_test.wsgi

#### デプロイ
まずherokuにログインして、SSHKeyを登録します。  
(nitrous ioにはHerokuのツールが標準でインストールされています)

    $heroku login
    email アカウント作成時のもの
    password 同上
    Authentication successful.
    $heroku key:add

Herokuにアプリを作成します。  
作成した時にアプリのurlが表示されるのでメモっておきます。

    $ heroku create                                                                                            
    Creating アプリ名... done, stack is cedar-14                                                                                                                      
    https://アプリ名.herokuapp.com/ 
    |https://git.heroku.com/アプリ名.git 

Herokuへのデプロイはgitを使って行います。

    $git init
    $git add .
    $git commit -m 'init'
    $git push heroku master
最後の$git push heroku masterでHerokuに対してコミットされます。
そこからは勝手にHerokuがPythonと必要なパッケージをインストールしてくれて、勝手にデプロイされます。
  
最後の最後にさっきメモったURLをブラウザで開いてみて「Hello Nitrous & Heroku」と出れば完成です。

参考URL  
https://devcenter.heroku.com/articles/getting-started-with-django  
http://help.nitrous.io/heroku/  
http://help.nitrous.io/django-app/  


