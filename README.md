implementation 'com.squareup.retrofit2:retrofit:2.3.0'

implementation 'com.squareup.retrofit2:converter-gson:2.3.0'


*****************************************************************************************************

implementation 'com.squareup.retrofit2:retrofit:2.3.0'

implementation 'com.squareup.retrofit2:converter-gson:2.3.0'

implementation 'com.squareup.retrofit2:converter-scalars:2.3.0'


public class ApiClient {

    private static String BASE_URL = "https://lochawala.com/foodplus/api/public/index.php/v1/";
    private static Retrofit retrofit = null;

    public static Retrofit getRetrofitClient(Context context) {

        if (retrofit == null) {
            retrofit = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create())
                    .build();
        }
        return null;
    }

}


***********************************************************************************************************

public interface ApiInterface {

    @POST("register.php")
    Call<ResponseBody> registerUser(@Body RequestBody body);
    
    ******************************************************************************
    
    @Headers("Content-Type: application/json")
    @POST("register")
    Call<ResponseBody> registerUser(@Body String body);
    
    @Headers("Content-Type: application/json")
    @POST("login")
    Call<ResponseBody> loginUser(@Body String body);
    
 }

***************************************************************************************************************

mApiService = ApiClient.getRetrofitClient(SplashActivity.this).create(ApiInterface.class);


*****************************************************************************************************************

ResponseBody mUser = response.body();
                                String mResponse = mUser != null ? mUser.string() : null;

                                JSONObject object = new JSONObject(mResponse);


*****************************************************************************************************
    if (mApiService != null && !TextUtils.isEmpty(mImei) && !TextUtils.isEmpty(mFcmToken)) {

            RequestBody requestBody = new MultipartBody.Builder()
                    .setType(MultipartBody.FORM)
                    .addFormDataPart("deviceID", mImei)
                    .addFormDataPart("device", "Android")
                    .addFormDataPart("fcm", mFcmToken)
                    .addFormDataPart("version", Common.API_VERSION)
                    .build();

            mApiService.registerUser(requestBody).enqueue(new Callback<ResponseBody>() {
                @Override
                public void onResponse(@NonNull Call<ResponseBody> call, @NonNull Response<ResponseBody> response) {

                    if (response.isSuccessful()) {

                        try {
                            ResponseBody mUser = response.body();
                            String mResponse = mUser != null ? mUser.string() : null;

                            JSONObject object = new JSONObject(mResponse);
                            JSONArray Jarray = object.getJSONArray("data");
                            for (int i = 0; i < Jarray.length(); i++) {
                                mUserID = Jarray.getJSONObject(i).getString("id");
                                Log.e("TAG", "===============>> userId: " + mUserID);
                            }
                            SharedPreferences sharedPref = getSharedPreferences(Config.SHARED_PREF, Context.MODE_PRIVATE);
                            SharedPreferences.Editor editor = sharedPref.edit();
                            editor.putString("imeino", mImei);
                            editor.putString("userid", mUserID);
                            editor.apply();

                            if (!TextUtils.isEmpty(mImei) && !TextUtils.isEmpty(mUserID)) {

                                loginToApp();
                            }

                        } catch (IOException | JSONException e) {
                            e.printStackTrace();
                        }

                    } else {
                        switch (response.code()) {
                            case 404:
                                Toast.makeText(SplashActivity.this, "not found", Toast.LENGTH_LONG).show();
                                break;
                            case 500:
                                Toast.makeText(SplashActivity.this, "server broken", Toast.LENGTH_LONG).show();
                                break;
                            default:
                                Toast.makeText(SplashActivity.this, "unknown error", Toast.LENGTH_LONG).show();
                                break;
                        }
                    }
                }

                @Override
                public void onFailure(@NonNull Call<ResponseBody> call, @NonNull Throwable t) {
                    Toast.makeText(SplashActivity.this, "No Internet Connection, Failed to connect with server", Toast.LENGTH_LONG).show();
                }
            });
        } else {
            new Handler().postDelayed(new Runnable() {
                @Override
                public void run() {
                    registerUser();
                }
            }, 300);
        }
        
        
        
        *****************************************************************************************************************
        
        
        
        
        
        
        
        
        
        private void _callApiLogin() {
        try {
            Common.showProgress(context, "Please wait");

            if (mApiService != null) {

                String userId = "";
                if (!PrefUtil.getstringPref(getString(R.string.pref_unique_id), context).equals("") && PrefUtil.getstringPref(getString(R.string.pref_unique_id), context) != null) {
                    userId = PrefUtil.getstringPref(getString(R.string.pref_unique_id), context);
                } else {
                    userId = "4444444444";
                }
                JSONObject paramObject = new JSONObject();
                paramObject.put("mobile_no", et_phone.getText().toString());
                paramObject.put("password", et_password.getText().toString());
                paramObject.put("user_id", userId);

                mApiService.loginUser(paramObject.toString()).enqueue(new Callback<ResponseBody>() {
                    @Override
                    public void onResponse(@NonNull Call<ResponseBody> call, @NonNull retrofit2.Response<ResponseBody> response) {
                        Common.dismissProgress();
                        if (response.isSuccessful()) {
                            try {
                                ResponseBody mUser = response.body();
                                String mResponse = mUser != null ? mUser.string() : null;

                                JSONObject object = new JSONObject(mResponse);
                                boolean status = object.getBoolean("status");
                                if (status) {
                                    JSONObject dataObject = object.getJSONObject("data");
                                    User userModel = new User();
                                    userModel.setUserId(dataObject.getInt("id"));
                                    userModel.setStatus(dataObject.getInt("status"));
                                    userModel.setMobileNo(dataObject.getString("mobile_no"));
                                    if (userModel.getStatus() == 0) {
                                        SnackBar.makeShort(context, getString(R.string.not_register));
                                    } else if (userModel.getStatus() == 1) {
                                        PrefUtil.putstringPref(getString(R.string.pref_user_id), String.valueOf(userModel.getUserId()), context);
                                        PrefUtil.putstringPref(getString(R.string.pref_phone), userModel.getMobileNo(), context);
                                        PrefUtil.putbooleanPref(getString(R.string.pref_is_login), true, context);
                                        if (isOther) {
                                            onBackPressed();
                                        } else {
                                            startActivity(new Intent(context, HomeActivity.class).addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK | Intent.FLAG_ACTIVITY_NEW_TASK));
                                        }
                                    }
                                }
                            } catch (Exception e) {
                                e.printStackTrace();
                            }
                        } else {
                            if (response.code() == 401) {
                                try {
                                    ResponseBody mUser = response.errorBody();
                                    String mResponse = mUser != null ? mUser.string() : null;

                                    JSONObject obj = new JSONObject(mResponse);
                                    boolean success = obj.getBoolean("success");
                                    String message = obj.getString("message");
                                    if (!success) {
                                        SnackBar.makeShort(context, message);
//                                        SnackBar.makeShort(context, getString(R.string.not_register));
                                    }
                                } catch (Exception e) {
                                    e.printStackTrace();
                                }
                            } else {
                                _callApiLogin();
                            }
                        }
                    }

                    @Override
                    public void onFailure(@NonNull Call<ResponseBody> call, @NonNull Throwable t) {
                        Common.dismissProgress();
                        SnackBar.makeShort(context, getString(R.string.error_something_wrong));
                    }
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
            Common.dismissProgress();
        }
    }
        
        
     *******************************************************************************************
     
     private void _callApiSignUp() {
        try {
            Common.showProgress(context, "Please wait");

            if (mApiService != null) {

                JSONObject paramObject = new JSONObject();
                paramObject.put("mobile_no", et_phone.getText().toString());

                mApiService.registerUser(paramObject.toString()).enqueue(new Callback<ResponseBody>() {
                    @Override
                    public void onResponse(@NonNull Call<ResponseBody> call, @NonNull retrofit2.Response<ResponseBody> response) {
                        Common.dismissProgress();
                        if (response.isSuccessful()) {
                            try {
                                ResponseBody mUser = response.body();
                                String mResponse = mUser != null ? mUser.string() : null;

                                JSONObject object = new JSONObject(mResponse);
                                boolean status = object.getBoolean("status");
                                String message = object.getString("message");
                                if (status) {
                                    JSONObject dataObject = object.getJSONObject("data");
                                    User userModel = new User();
                                    userModel.setUserId(dataObject.getInt("id"));
                                    userModel.setOtp(dataObject.getString("request_otp"));//todo remove line
                                    userModel.setStatus(dataObject.getInt("status"));
                                    userModel.setMobileNo(dataObject.getString("mobile_no"));
                                    Toast.makeText(context, "OTP: " + userModel.getOtp(), Toast.LENGTH_SHORT).show();//todo

                                    if (userModel.getStatus() == 0) {
                                        startActivity(new Intent(context, OtpActivity.class).putExtra(getString(R.string.intent_isSignUp), true).putExtra(getString(R.string.intent_user_model), userModel).putExtra(getString(R.string.intent_isOther), isOther));
                                    } else if (userModel.getStatus() == 1) {
                                        SnackBar.makeShort(context, getString(R.string.already_register));
                                    }
                                } else {
                                    SnackBar.makeShort(context, message);
                                }
                            } catch (Exception e) {
                                e.printStackTrace();
                            }
                        } else {
                            if (response.code() == 401) {
                                try {
                                    ResponseBody mUser = response.errorBody();
                                    String mResponse = mUser != null ? mUser.string() : null;

                                    JSONObject obj = new JSONObject(mResponse);
                                    JSONObject msgObject = obj.getJSONObject("message");
                                    JSONArray array = msgObject.getJSONArray("mobile_no");
                                    for (int i = 0; i < array.length(); i++) {
                                        String message = array.getString(i);
                                        SnackBar.makeShort(context, message);
                                    }
                                } catch (Exception e) {
                                    e.printStackTrace();
                                }
                            } else {
                                _callApiSignUp();
                            }
                        }
                    }

                    @Override
                    public void onFailure(@NonNull Call<ResponseBody> call, @NonNull Throwable t) {
                        Common.dismissProgress();
                        SnackBar.makeShort(context, getString(R.string.error_something_wrong));
                    }
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
            Common.dismissProgress();
        }
    }
        
        
        
        
        
