
01 Amb操作符可以将至多9个Observable结合起来，让他们竞争。哪个Observable首先发射了数据（包括onError和onComplete)就会继续发射这个Observable的数      据，其他的Observable所发射的数据都会被丢弃 <br>

    Observable<Integer> observable1 = Observable.just(1,2,3).delay(3000,TimeUnit.MILLISECONDS);
    Observable<Integer> observable2 = Observable.just(4,5,6).delay(1000,TimeUnit.MILLISECONDS);
    Observable<Integer> observable3 = Observable.just(7,8,9).delay(2000,TimeUnit.MILLISECONDS);
    Observable.amb(observable1,observable2,observable3)
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG,"onNext" + integer);

            }
        });
        
     打印结果是onNext 4,5,6
02 combineLatest是RxJava本身提供的一个常用的操作符，它接受两个或以上的Observable和一个FuncX闭包。当传入的Observable中任意的一个发射数据时，        combineLatest将每个Observable的最近值(Lastest)联合起来（combine）传给FuncX闭包进行处理。要点在于 <br>
   <1> combineLatest是会存储每个Observable的最近的值的 <br>
   <2> 任意一个Observable发射新值时都会触发操作-><br>
   用combineLatest处理表单验证 首先我们写上email和password的验证方法，一个需要含有@字符，一个要求字符数超过4个 <br>
     
    private boolean isEmailValid(String email) {
        //TODO: Replace this with your own logic
        return email.contains("@");
    }

    private boolean isPasswordValid(String password) {
    //TODO: Replace this with your own logic
    return password.length() > 4;
    }
   
   随后，我们针对email和password分别创建Observable，发射的值即为各自edittext的变化的内容，
   而call回调方法的返回值是textWatcher中afterTextChanged   方法的传入参数： <br>
    
    
      Observable<String> ObservableEmail = Observable.create(new Observable.OnSubscribe<String>() {

            @Override
            public void call(final Subscriber<? super String> subscriber) {
                mEmailView.addTextChangedListener(new TextWatcher() {
                    @Override
                    public void beforeTextChanged(CharSequence s, int start, int count, int after) {

                    }

                    @Override
                    public void onTextChanged(CharSequence s, int start, int before, int count) {

                    }

                    @Override
                    public void afterTextChanged(Editable s) {
                        subscriber.onNext(s.toString());
                    }
                });
            }
        });

    Observable<String> ObservablePassword = Observable.create(new Observable.OnSubscribe<String>() {

      @Override
      public void call(final Subscriber<? super String> subscriber) {
        mPasswordView.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {

            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {

            }

            @Override
            public void afterTextChanged(Editable s) {
                subscriber.onNext(s.toString());
            }
        });
      }
    });
   最后，用combineLastest将ObservableEmail和ObservablePassword联合起来进行验证：<br>

      Observable.combineLatest(ObservableEmail, ObservablePassword, new Func2<String, String, Boolean>() {
            @Override
            public Boolean call(String email, String password) {
                return isEmailValid(email) && isPasswordValid(password);
            }
        }).subscribe(new Subscriber<Boolean>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(Boolean verify) {
                if (verify) {
                    mEmailSignInButton.setEnabled(true);
                } else {
                    mEmailSignInButton.setEnabled(false);
                }
            }
        });


