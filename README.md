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
