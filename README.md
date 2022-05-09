# appCurrency
private class GetRbcExchangeRates extends AsyncTask<String, String, String> {

    public ArrayList<CurrencyRateModel> listOfCurrency = new ArrayList<>();
    private final String httpAddress = "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json";

    @Override
    protected String doInBackground(String... params) {

        HttpURLConnection connection = null;
        BufferedReader bufferedReader = null;
        StringBuilder jSonResult = new StringBuilder();

        //listOfCurrency = new ArrayList<>();

        try {
            URL url = new URL(httpAddress);
            connection = (HttpURLConnection)url.openConnection();
            connection.connect();

            bufferedReader = new BufferedReader(new InputStreamReader(connection.getInputStream()));

            String line = "";

            while((line = bufferedReader.readLine()) != null) {

                jSonResult.append(line);

            }


        }
        catch(Exception e) {
            e.printStackTrace();
            publishProgress("MESSAGE", "Error 404");
        }
        finally {

            if(connection != null)
                connection.disconnect();

            try {
                if(bufferedReader !=null)
                    bufferedReader.close();
            }
            catch(IOException e) {
                e.printStackTrace();
            }

        }

        try {

            JSONObject rbcGETRates = new JSONObject(jSonResult.toString());
            JSONObject jSonValute = rbcGETRates.getJSONObject("Valute");
            Iterator<String> arrayKey = jSonValute.keys();

            while(arrayKey.hasNext()) {

                String key = arrayKey.next();
                JSONObject jSonItem = jSonValute.getJSONObject(key);
                CurrencyRateModel currencyitems = new CurrencyRateModel();
                currencyitems.id = jSonItem.getString("ID");
                currencyitems.nameCode = jSonItem.getString("NumCode");
                currencyitems.charCode = jSonItem.getString("CharCode");
                currencyitems.nominal = jSonItem.getInt("Nominal");
                currencyitems.name = jSonItem.getString("Name");
                currencyitems.value = jSonItem.getDouble("Value");
                currencyitems.previous = jSonItem.getDouble("Previous");
                listOfCurrency.add(currencyitems);
            }
        }
        catch (JSONException e) {

            e.printStackTrace();
        }

        return null;
    }

    @Override
    protected void onPreExecute() {
        super.onPreExecute();

        ListView lsView = (ListView)findViewById(R.id.currencyView);
        MyAdapter myAdapter = new MyAdapter(MainActivity.this, listOfCurrency);
        lsView.setAdapter(myAdapter);
    }

    @Override
    protected void onProgressUpdate(String... values) {
        super.onProgressUpdate(values);
        if(values[0].equals("MESSAGE")) {
            Toast.makeText(MainActivity.this, values[1], Toast.LENGTH_LONG).show();
        }
    }

    @Override
    protected void onPostExecute(String s) {
        super.onPostExecute(s);
    }
}
public class CurrencyRateModel {

public String id;
public String nameCode;
public String charCode;
public int nominal;
public String name;
double value;
double previous;
}
public class MyAdapter extends BaseAdapter {

private Context ctx;
private ArrayList<CurrencyRateModel> arrayList;
private LayoutInflater inflater;

public MyAdapter(Context ctx, ArrayList<CurrencyRateModel> arrayList) {

    this.ctx = ctx;
    this.arrayList = arrayList;
    inflater = ((Activity)ctx).getLayoutInflater();
}

@Override
public int getCount() {
    return arrayList.size();
}


@Override
public CurrencyRateModel getItem(int position) {
   return arrayList.get(position);
}

@Override
public long getItemId(int position) {
    return position;
}

@Override
public View getView(int position, View convertView, ViewGroup parent) {

    ViewHolder vh;

    if(convertView == null) {
        
        vh = new ViewHolder();
      
        convertView = inflater.inflate(R.layout.itemcurrency, null);
        
        vh.tvTitle = (TextView)convertView.findViewById(R.id.title);
  
        vh.tvDescription = (TextView)convertView.findViewById(R.id.description);
       
        convertView.setTag(vh);

    }
    else {
        
        vh = (ViewHolder) convertView.getTag();
    }

   
    vh.tvTitle.setText(arrayList.get(position).name);
    vh.tvDescription.setText(Double.toString(arrayList.get(position).value));

    return convertView;
}


 @Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    // async TASK Run
    GetNBUExchangeRates async = new GetNBUExchangeRates();
    async.execute();
}
<TextView
    android:id="@+id/title"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textSize="22sp"/>
<TextView
    android:id="@+id/description"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:textSize="18sp"
    android:textColor="#555"/>
