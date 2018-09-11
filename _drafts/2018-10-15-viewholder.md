---
layout: post
title: "ViewHolder"
date:  2018-10-15
categories: ["Wzorce projektowe"]
image: viewholder
github: design-patterns/tree/master/viewholder
description: "Wzorce projektowe / kreacyjny"
keywords: "viewholder, wzorzec, wzorce projektowe, wzorzec kreacyjny, design patterns, android, java, programowanie, programming"
---

## Zastosowanie
`ViewHolder` (ang. `ViewHolder`) (wzorzec kreacyjny) ma za zadanie zoptymalizować proces renderowania widoku kolekcji elementów poprzez ponowne użycie obiektów istniejących widoków. W trakcie przewijania listy dla każdego elementu kolekcji wywoływana jest kosztowna operacja wiązania widoku `findViewById`. W efekcie wzrasta zużycie zasobów, powstaje więcej obiektów widoku niż to w danej chwili potrzebne, a także przewijanie listy nie jest płynne. `ViewHolder` likwiduje wspomniane problemy. Przechowuje referencję do związanych instancji widoku w postaci obiektu typu `ViewHolder` i w razie potrzeby udostępnia je do ponownego użycia. Ilość stworzonych obiektów `ViewHolder` jest zależna od wielkości ekranu urządzenia i wynosi mniej więcej tyle ile maksymalnie może zostać wyświetlonych elementów w widoku (naraz bez przewijania).

## Ograniczenia
Wykorzystanie wzorca `ViewHolder` nieznacznie zwiększa poziom skomplikowania.

## Użycie
Wzorzec używany jest w widokach kolekcji w celu optymalizacji zużycia zasobów pamięci urządzenia. Dla niewielkich kolekcji (mieszczących się w całości na ekranie) różnica w wydajności może nie występować lub być niezauważalna, jednakże gdy nie mieszczą się one na ekranie i możliwe jest przewinięcie listy wówczas skok wydajności jest zauważalnym gołym okiem.

## Implementacja
Wewnętrzna klasa `ViewHolder` przechowuje referencję do obiektów widoku `View`. Referencje są ustawiane w klasie adaptera bezpośrednio na obiekcie viewholder lub przekazywany jest cały widok do konstruktora `ViewHolder`, który sam dba o związanie obiektów widoku. Klasa `Adapter` wykorzystuje obiekt typu `ViewHolder` do pobrania referencji i przekazania danych do widoków.

![ViewHolder diagram](/assets/img/diagrams/viewholder.svg){: .center-image }

W celu uproszczenia idei wzorca `ViewHolder` przedstawiona poniżej implementacja dedykowania jest dla widoku kolekcji typu `ListView` oraz klasy adaptera rozszerzającego `BaseAdapter`. Jednakże zalecane jest wykorzystanie widoku kolekcji `RecyclerView` wraz implementacją klasy rozszerzającej `RecyclerView.Adapter`.

{% highlight java %}
public class ItemAdapter extends BaseAdapter {

    private List<Item> items;

    public ItemAdapter(List<Item> items) {
        this.items = items;
    }

    class ViewHolder {

        private TextView textView;
        private ImageView imageView;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;

        //check if received view has been inflated before
        if(convertView == null) {
            //get layout to inflate
            LayoutInflater inflater = LayoutInflater.from(getContext());
            convertView = inflater.inflate(R.layout.item, parent, false);
            viewHolder = viewHolder(convertView);

            //inflate views into viewHolder
            viewHolder.textView = convertView.findViewById(R.id.textView);
            viewHolder.imageView = convertView.findViewById(R.id.imageView);
            convertView.setTag(viewHolder);
        }
        else {
            //receive inflated view from memory
            viewHolder = (ViewHolder) convertView.getTag();
        }

        //set data into views
        Item item = items.get(position);
        viewHolder.textView.setText(item.getText());
        viewHolder.imageView.seImageResource(item.getImage());

        return convertView;
    }

    //here should be other adapter methods override from superclass
    @Override
    public int getCount() {
        return items.size();
    }

    @Override
    public Item getItem(int position) {
        return item.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }
}
{% endhighlight %}

Aktywność lub fragment tworzą tworzą widok listy do którego przekazany jest adapter z danymi kolekcji wykorzystujący `ViewHolder`.

{% highlight java %}
//some activity or fragment lifecycle methods
//this work can be done in e.g. in onCreate or onCreateView

//create or get some collection
List<Item> items = new ArrayList<>();
items.add(new Item("Item1", R.drawable.image1));
items.add(new Item("Item2", R.drawable.image2));
items.add(new Item("Item3", R.drawable.image3));
//more and more items

//insert adapter with ViewHolder into ListView
ListView listView = findViewById(R.id.listView);
ItemAdapter adapter = new ItemAdapter(items);
listView.setAdapter(adapter);
{% endhighlight %}

## Przykład
Aplikacja `FoodDeliver` umożliwia użytkownikom dokonanie zamówienia dostawy jedzenia z wybranej restauracji. Wiele widoków aplikacji (w tym widok pozycji menu z danej restauracji) wykorzystuje mechanizm widoku przewijalnej listy. W celu optymalizacji wydajności działania listy do realizacji tego zadania użyto kontrolkę `RecyclerView` wraz z implementacją `ViewHolder`.

{% highlight java %}

public class FoodAdapter extends RecyclerView.Adapter<FoodAdapter.ViewHolder> {

    private Context context;
    private List<Food> items;

    public FoodAdapter(Context context, List<Food> items) {
        this.context = context;
        this.items = items;
    }

    class ViewHolder extends RecyclerView.ViewHolder {

        private ImageView imageView;
        private TextView nameView;
        private TextView priceView;
        private Button cartButton;

        ViewHolder(View view) {
            super(view);
            imageView = view.findViewById(R.id.food_image);
            nameView = view.findViewById(R.id.food_name);
            priceView = view.findViewById(R.id.food_price);
            cartButton = view.findViewById(R.id.food_cart);
        }

        public void setData(Food item) {
            Picasso.with(context).load(item.getImageUrl()).into(imageView);
            nameView.setText(item.getName());
            priceView.setText(item.getPrice() + " PLN");
            cartButton.setOnClickListener(v -> addToCart(item));
        }
    }

    @Override
    public FoodAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.food_item, parent, false);
        return new FoodAdapter.ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull FoodAdapter.ViewHolder holder, int position) {
        Food item = items.get(position);
        holder.setData(item);
    }

    @Override
    public int getItemCount() {
        return items.size();
    }

    private void addToCart(Food item) {
        //add dish to cart action
    }
}
{% endhighlight %}

Aktywność lub fragment pobierają dane z serwera o dostępnych pozycjach menu. Następnie tworzony jest adapter, który jest przekazywany do widoku kolekcji `RecyclerView`.

{% highlight java %}
//user chose restaurant and goes to dishes selection screen
//app receives data from server for choosen restaurant and load into items collection
List<Food> items = new ArrayList<>();
items.add("Pizza Margherita", 15.00, "http://example.com/margherita.jpg");
items.add("Pizza Pepperoni", 18.00, "http://example.com/pepperoni.jpg");
items.add("Pizza Hawaii", 20.00, "http://example.com/hawaii.jpg");
//and more more items

//inflate layout and other lifecycle stuff
RecyclerView recyclerView = findViewById(R.id.recyclerView);

//insert adapter into recyclerView
LinearLayoutManager layoutManager = new LinearLayoutManager(getActivity());
recyclerView.setLayoutManager(layoutManager);
FoodAdapter adapter = new FoodAdapter(getActivity(), items);
recyclerView.setAdapter(adapter);

//scrolling the list is smooth and faster
{% endhighlight %}

## Biblioteki
Użycie widoku typu `RecyclerView` wymusza na programiście implementację wewnętrznej klasy `RecyclerView.ViewHolder`. Realizacja wzorca w starszych widokach kolekcji np.: `ListView`, `GridView` jest opcjonalna.