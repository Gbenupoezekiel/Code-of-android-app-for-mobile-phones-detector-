# Code-of-android-app-for-mobile-phones-detector-
package com.gbenupoezekiel.examsCapture;

import android.Manifest;
import android.app.AlertDialog;
import android.app.ProgressDialog;
import android.bluetooth.BluetoothAdapter;
import android.bluetooth.BluetoothDevice;
import android.bluetooth.BluetoothSocket;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.ActivityInfo;
import android.content.pm.PackageManager;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.location.Criteria;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.media.MediaPlayer;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.AsyncTask;
import android.os.Build;
import android.os.Environment;
import android.provider.Settings;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.ContextCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Base64;
import android.view.View;
import android.view.WindowManager;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ImageView;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Set;
import java.util.UUID;

public class MainActivity extends AppCompatActivity implements LocationListener {
    //
    private BluetoothAdapter BA;
    private OutputStream outputStream;
    private InputStream inputStream;
    private BluetoothSocket socket;
    private BluetoothDevice bluetoothDevice;
    ImageView connectDevice;
    //
    Context context;
    Intent intent1;
    Location location;
    LocationManager locationManager;
    boolean GpsStatus = false;
    Criteria criteria;
    String Holder;
    String latitude;
    String longitude;
    String currentlocation;
    public static final int REQUEST_ID_MULTIPLE_PERMISSIONS=1;
    //
    WebView webView;
    TextView measuredDistance;
    //
    private Bitmap bitmap;
    //
    public static final String UPLOAD_URL = "https://www.akifagoelectronics.com.ng/I@M5A9G5E@P8H0P/techUpload.php";
    public static final String UPLOAD_KEY = "images";
    //
    Thread workerThread;
    byte[]readBuffer;
    int readBufferPosition;
    volatile boolean stopWorker;
    int bluetoothstatus=0;
    //
    MediaPlayer pleaseconnect;
    MediaPlayer bluetoothdisconnected;
    MediaPlayer bluetoothconnected;
    MediaPlayer visitoralert;
    MediaPlayer internetalert;
    //
    int fullScreen = 0;
    int fullScreenCount = 0;
    //
    ListView bluetoothlist;
    //The BroadcastReceiver that listens for bluetooth broadcasts
    private final BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();

            if (BluetoothDevice.ACTION_ACL_CONNECTED.equals(action)) {
                //Do something if connected
                bluetoothstatus=1;
                bluetoothconnected.start();
            }
            if (BluetoothDevice.ACTION_ACL_DISCONNECT_REQUESTED.equals(action)){
                Toast.makeText(getApplicationContext(), "Please Pair Bluetooth Device", Toast.LENGTH_SHORT).show();
            }
            else if (BluetoothDevice.ACTION_ACL_DISCONNECTED.equals(action)) {
                //Do something if disconnected
                bluetoothstatus=0;
                bluetoothdisconnected.start();
            }
        }
    };
    //
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //
        getWindow().addFlags(
                WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON |
                        WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD |
                        WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED |
                        WindowManager.LayoutParams.FLAG_TURN_SCREEN_ON);
        //
        setContentView(R.layout.activity_main);
        //
        setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
        //
        //Permission check
        List<String> listPermissionsNeeded = new ArrayList<String>();
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (ContextCompat.checkSelfPermission(MainActivity.this,Manifest.permission.ACCESS_COARSE_LOCATION)!=PackageManager.PERMISSION_GRANTED){
                listPermissionsNeeded.add(Manifest.permission.ACCESS_COARSE_LOCATION);
            }
            if (ContextCompat.checkSelfPermission(MainActivity.this,Manifest.permission.ACCESS_FINE_LOCATION)!=PackageManager.PERMISSION_GRANTED){
                listPermissionsNeeded.add(Manifest.permission.ACCESS_FINE_LOCATION);
            }
            if (ContextCompat.checkSelfPermission(MainActivity.this,Manifest.permission.INTERNET)!=PackageManager.PERMISSION_GRANTED){
                listPermissionsNeeded.add(Manifest.permission.INTERNET);
            }
            if (ContextCompat.checkSelfPermission(MainActivity.this,Manifest.permission.WRITE_EXTERNAL_STORAGE)!=PackageManager.PERMISSION_GRANTED){
                listPermissionsNeeded.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);
            }
            if (!listPermissionsNeeded.isEmpty()){
                ActivityCompat.requestPermissions(MainActivity.this,listPermissionsNeeded.toArray(new String[listPermissionsNeeded.size()]),REQUEST_ID_MULTIPLE_PERMISSIONS);
            }
        }
        //
        IntentFilter filter1 = new IntentFilter(BluetoothDevice.ACTION_ACL_CONNECTED);
        IntentFilter filter2 = new IntentFilter(BluetoothDevice.ACTION_ACL_DISCONNECT_REQUESTED);
        IntentFilter filter3 = new IntentFilter(BluetoothDevice.ACTION_ACL_DISCONNECTED);
        this.registerReceiver(mReceiver, filter1);
        this.registerReceiver(mReceiver, filter2);
        this.registerReceiver(mReceiver, filter3);
        //
        pleaseconnect = MediaPlayer.create(this,R.raw.pleaseconnect);
        bluetoothdisconnected = MediaPlayer.create(this,R.raw.btdisconnected);
        bluetoothconnected = MediaPlayer.create(this,R.raw.btconnected);
        visitoralert = MediaPlayer.create(this,R.raw.visitor);
        internetalert = MediaPlayer.create(this,R.raw.warning);
        //
        bluetoothlist = (ListView)findViewById(R.id.listView);
        webView = (WebView)findViewById(R.id.webView);
        connectDevice = (ImageView)findViewById(R.id.imageView);
        measuredDistance = (TextView)findViewById(R.id.textView);
        BA = BluetoothAdapter.getDefaultAdapter();
        BA.enable();
        //
        connectDevice.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //List Bluetooth Devices
                Set<BluetoothDevice> pairedDevices = BA.getBondedDevices();
                ArrayList<String> list = new ArrayList<>();
                for (BluetoothDevice bt : pairedDevices) {
                    list.add(bt.getName());
                    final ArrayAdapter<String> adapter = new ArrayAdapter<>(MainActivity.this, android.R.layout.simple_list_item_1, list);
                    bluetoothlist.setAdapter(adapter);
                    bluetoothlist.setVisibility(View.VISIBLE);
                }
            }
        });
        //
        //Get GPS Location
        //
        locationManager = (LocationManager)getSystemService(Context.LOCATION_SERVICE);
        criteria = new Criteria();
        Holder = locationManager.getBestProvider(criteria, false);
        context = getApplicationContext();
        CheckGpsStatus();
        //
        if (!locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER)) {
            intent1 = new Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS);
            startActivity(intent1);
        }
        //
        if (GpsStatus) {
            if (Holder != null) {
                location = locationManager.getLastKnownLocation(Holder);
                locationManager.requestLocationUpdates(Holder, 12000, 7, MainActivity.this);
            }
        }else {
            Toast.makeText(getBaseContext(),"Please Enable GPS Location Service First",Toast.LENGTH_LONG).show();
        }
        //
        bluetoothlist.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                String selecteddevice = ((TextView)view).getText().toString();
                //
                Set<BluetoothDevice> pairedDevices=BA.getBondedDevices();
                if (pairedDevices.size()>0){
                    for (BluetoothDevice device:pairedDevices){
                        if (device.getName().equals(selecteddevice)){
                            bluetoothDevice  = device;
                            UUID uuid = UUID.fromString("00001101-0000-1000-8000-00805F9B34FB");
                            try {
                                socket = bluetoothDevice.createRfcommSocketToServiceRecord(uuid);
                                socket.connect();
                                outputStream = socket.getOutputStream();
                                inputStream = socket.getInputStream();
                                bluetoothstatus=1;
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
                //Reads Incoming Bluetooth Information and Display on Bluetooth Info
                if (BA!=null&&BA.isEnabled() && bluetoothstatus==1) {
                    final android.os.Handler handler = new android.os.Handler();
                    final byte delimiter = 10;
                    stopWorker = false;
                    readBufferPosition = 0;
                    readBuffer = new byte[1024];
                    workerThread = new Thread(new Runnable() {
                        @Override
                        public void run() {
                            while (!Thread.currentThread().isInterrupted() && !stopWorker) {
                                try {
                                    int bytesAvailable = inputStream.available();
                                    if (bytesAvailable > 0) {
                                        byte[] packetBytes = new byte[bytesAvailable];
                                        inputStream.read(packetBytes);
                                        for (int i = 0; i < bytesAvailable; i++) {
                                            byte b = packetBytes[i];
                                            if (b == delimiter) {
                                                byte[] encodedBytes = new byte[readBufferPosition];
                                                System.arraycopy(readBuffer, 0, encodedBytes, 0, encodedBytes.length);
                                                final String data = new String(encodedBytes, "US-ASCII");
                                                readBufferPosition = 0;
                                                handler.post(new Runnable() {
                                                    @Override
                                                    public void run() {
                                                        Toast.makeText(getBaseContext(), data, Toast.LENGTH_LONG).show();
                                                        if (data.contains(",")){
                                                            String[] deviceCoordinate = data.split(",");
                                                            String deviceLat = deviceCoordinate[0];
                                                            String deviceLng = deviceCoordinate[1];
                                                            int Radius = 6371;
                                                            double lat1 = Double.parseDouble(latitude);
                                                            double lat2 = Double.parseDouble(deviceLat);
                                                            double lon1 = Double.parseDouble(longitude);
                                                            double lon2 = Double.parseDouble(deviceLng);
                                                            double dLat = Math.toRadians(lat2 - lat1);
                                                            double dLon = Math.toRadians(lon2 - lon1);
                                                            double a = Math.sin(dLat/2) * Math.sin(dLat/2)
                                                                    + Math.cos(Math.toRadians(lat1))
                                                                    * Math.cos(Math.toRadians(lat2)) * Math.sin(dLon/2)
                                                                    * Math.sin(dLon/2);
                                                            double c = 2 * Math.asin(Math.sqrt(a));
                                                            double valueResult = Radius * c;
                                                            double km = valueResult/1;
                                                            DecimalFormat newFormat = new DecimalFormat("####");
                                                            int kmInDec = Integer.valueOf(newFormat.format(km));
                                                            double meter = valueResult % 1000;
                                                            int meterInDec = Integer.valueOf(newFormat.format(meter));
                                                            //
                                                            if (kmInDec > 0 || meterInDec > 0) {
                                                                measuredDistance.setText("Distance: " + meterInDec + " m");
                                                            }
                                                            else{
                                                                double l1 = Math.toRadians(lat1);
                                                                double l2 = Math.toRadians(lat2);
                                                                double g1 = Math.toRadians(lon1);
                                                                double g2 = Math.toRadians(lon2);
                                                                double dist = Math.acos(Math.sin(l1) * Math.sin(l2) + Math.cos(l1) * Math.cos(l2) * Math.cos(g1 - g2));
                                                                if (dist < 0){
                                                                    dist = dist + Math.PI;
                                                                }
                                                                dist = Math.round(dist * 6378100);
                                                                if (dist > 0){
                                                                    measuredDistance.setText("Distance: " + dist + " cm");
                                                                }
                                                                else{
                                                                    float[]results = new float[1];
                                                                    Location.distanceBetween(lat1, lon1, lat2, lon2, results);
                                                                    measuredDistance.setText("Distance: " + results + " m");
                                                                }
                                                            }
                                                            //
                                                            takeScreenshot();
                                                            visitoralert.start();
                                                        }
                                                    }
                                                });
                                            } else {
                                                readBuffer[readBufferPosition++] = b;
                                            }
                                        }
                                    }
                                } catch (IOException e) {
                                    stopWorker = true;
                                }
                            }
                        }
                    });
                    workerThread.start();
                } else {
                    pleaseconnect.start();
                }
            }
        });
        //
        loadPage();
        //
    }

    public void loadPage(){
        webView.setInitialScale(1);
        webView.getSettings().setJavaScriptEnabled(true);
        webView.getSettings().setLoadWithOverviewMode(true);
        webView.getSettings().setUseWideViewPort(true);
        webView.setScrollBarStyle(WebView.SCROLLBARS_OUTSIDE_OVERLAY);
        webView.setScrollbarFadingEnabled(false);
        webView.loadUrl("file:///android_asset/espCamView.html");
        webView.setWebViewClient(new WebViewClient(){
            public void onReceivedError(final WebView webView, int errorCode, String description, String failingUrl){
                try{
                    webView.stopLoading();
                }catch (Exception e){ }
                if (webView.canGoBack()){
                    webView.goBack();
                }

                webView.loadUrl("about:blank");
                AlertDialog alertDialog = new AlertDialog.Builder(MainActivity.this).create();
                alertDialog.setTitle("Error");
                alertDialog.setMessage("No video feed found. Turn ON camera and try again.");
                alertDialog.setButton(DialogInterface.BUTTON_POSITIVE, "Try Again", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        finish();
                        startActivity(getIntent());
                    }
                });
                alertDialog.show();
                super.onReceivedError(webView, errorCode, description, failingUrl);
            }
        });
    }
    public void onBackPressed(){
        BA.disable();
        finish();
        System.exit(0);
    }
    private void takeScreenshot(){
        Bitmap b = Bitmap.createBitmap(webView.getWidth(), webView.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas c = new Canvas(b);
        webView.draw(c);
        File root = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM),"Akifago");
        if (!root.exists())
        {
            root.mkdirs();
            Toast.makeText(getBaseContext(),"Folder Does Not Exist.",Toast.LENGTH_SHORT).show();
        }
        String fname = String.format("%d.jpg",System.currentTimeMillis());
        File file = new File(root, fname);
        if (file.exists()) file.delete();
        try {
            FileOutputStream ostream = new FileOutputStream(file);
            Toast.makeText(getBaseContext(),"Saving Image...",Toast.LENGTH_SHORT).show();
            b.compress(Bitmap.CompressFormat.JPEG, 100, ostream);
            ostream.flush();
            ostream.close();
            Toast.makeText(getBaseContext(),"Image File Saved Successfully.",Toast.LENGTH_SHORT).show();
        } catch (Exception e) {
            e.printStackTrace();
        }
        //
        bitmap = b;
        //Checks Internet Connectivity
        ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        if (connectivityManager.getNetworkInfo(ConnectivityManager.TYPE_MOBILE).getState() == NetworkInfo.State.CONNECTED ||
                connectivityManager.getNetworkInfo(ConnectivityManager.TYPE_WIFI).getState() == NetworkInfo.State.CONNECTED) {
            //
            uploadImage();
            //
        } else {
            internetalert.start();
            Toast.makeText(getBaseContext(), "Please Make Sure Internet Connection is Available", Toast.LENGTH_LONG).show();
        }
    }
    //
    public String getStringImage(Bitmap bmp){
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        bmp.compress(Bitmap.CompressFormat.JPEG, 100, baos);
        byte[] imageBytes = baos.toByteArray();
        String encodedImage = Base64.encodeToString(imageBytes, Base64.DEFAULT);
        return encodedImage;
    }
    //
    private void uploadImage(){
        class UploadImage extends AsyncTask<Bitmap,Void,String> {

            ProgressDialog loading;
            RequestHandler rh = new RequestHandler();

            @Override
            protected void onPreExecute() {
                super.onPreExecute();
                loading = ProgressDialog.show(MainActivity.this, "Uploading...", null,true,true);
            }

            @Override
            protected void onPostExecute(String s) {
                super.onPostExecute(s);
                loading.dismiss();
                Toast.makeText(getApplicationContext(),s,Toast.LENGTH_LONG).show();
            }

            @Override
            protected String doInBackground(Bitmap... params) {
                Bitmap bitmap = params[0];
                String uploadImage = getStringImage(bitmap);

                HashMap<String,String> data = new HashMap<>();

                data.put(UPLOAD_KEY, uploadImage);
                String result = rh.sendPostRequest(UPLOAD_URL,data);

                return result;
            }
        }

        UploadImage ui = new UploadImage();
        ui.execute(bitmap);
    }
    //Get Current Location
    //
    @Override
    public void onLocationChanged(Location location) {
        latitude= String.valueOf(location.getLatitude());
        longitude= String.valueOf(location.getLongitude());
        currentlocation = "";
        currentlocation = latitude + "," + longitude;
        Toast.makeText(getBaseContext(), "Current GPS Location Found: " + currentlocation, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onStatusChanged(String s, int i, Bundle bundle) {

    }

    @Override
    public void onProviderEnabled(String s) {

    }

    @Override
    public void onProviderDisabled(String s) {

    }
    public void CheckGpsStatus(){
        locationManager = (LocationManager)context.getSystemService(Context.LOCATION_SERVICE);
        GpsStatus = locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER);
    }
    //
}
