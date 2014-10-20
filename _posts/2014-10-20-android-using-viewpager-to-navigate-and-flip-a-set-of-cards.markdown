---
layout: post
title: Android - Using ViewPager to Navigate and Flip a set of Cards
---

Recently I had to implement a series of "cards" in an App that a user
could swipe between and "flip over" to view more detail on each card. I
thought this use case may actually be rather common (flashcards, etc.)
so I've documented my technique here and whipped up a quick sample project.

The solution to this problem is to wrap the card back and front within a
containing `Fragment`, so that the `ViewPager` only deals with these
containers. This has the advantage of eliminating the need to
dynamically replace any Fragments through the `ViewPager` directly.

Right lets have a look at some code, here's how I have setup the main
`Activity` and adapter:

```
public class CardsActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_card);

        CardPagerAdapter adapter =
          new CardPagerAdapter(getFragmentManager());
        ViewPager viewPager =
          (ViewPager) findViewById(R.id.view_pager);
        viewPager.setAdapter(adapter);
    }

    public class CardPagerAdapter extends
      android.support.v13.app.FragmentPagerAdapter {

        public CardPagerAdapter(FragmentManager fm) { super(fm); }

        @Override
        public Fragment getItem(int i) {
            return new CardContainerFragment();
        }

        @Override
        public int getCount() {
            return 5;
        }
    }
}
```

Pretty simple so far, we've got an `Activity` to hold onto all of our
cards, a `ViewPager` within this to handle swiping back and forth between
cards, and an adapter to popular this ViewPager with a set of
`CardContainerFragment`.

As the name suggests, the `CardContainerFragment` class is just going to
act as a wrapper for our front and back fragments, so we can go ahead
and set this up with a `FrameLayout` to hold these:

```
public class CardContainerFragment extends Fragment {

    private boolean cardFlipped = false;

    public CardContainerFragment() {
        setHasOptionsMenu(true);
    }

    @Override
    public View onCreateView(LayoutInflater inflater,
                             ViewGroup container,
                             Bundle savedInstanceState) {
      View rootView = inflater.inflate(
        R.layout.fragment_card_container, container, false);

        getChildFragmentManager()
                .beginTransaction()
                .add(R.id.container, new CardFrontFragment())
                .commit();

        return rootView;
    }

    @Override
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        inflater.inflate(R.menu.card, menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch(item.getItemId()) {
            case R.id.action_flip:
                flipCard();
                return true;
        }

        return false;
        }
    }
}
```

You can see I've also added a `Menu` here which just holds a button to
trigger the card flipping. I'm also using a boolean flag on each
instance to determine whether the card is in the "flipped" state or not.
The layout for this fragment is similarly pretty simple:

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"></FrameLayout>

</LinearLayout>
```

All that's left to do is implement `flipCard()`:

```
public void flipCard() {
    Fragment newFragment;
    if (cardFlipped) {
        newFragment = new CardFrontFragment();
    } else {
        newFragment = new CardBackFragment();
    }

    getChildFragmentManager()
            .beginTransaction()
            .setCustomAnimations(
                R.animator.card_flip_right_in,
                R.animator.card_flip_right_out,
                R.animator.card_flip_left_in,
                R.animator.card_flip_left_out)
            .replace(R.id.container, newFragment)
            .commit();

    cardFlipped = !cardFlipped;
}
```

Here we are using the child `FragmentManager` provided by the container,
so all the logic of managing the front and back fragments is nicely
encapsulated within the `CardContainerFragment`. I've also added the
flip animation here, there is some official documentation on how to
achieve this [here](http://developer.android.com/training/animation/cardflip.html).

And here is the final result:

![CardViewPager Demo](/images/2014/10/20/card-pager-nexus.gif)


That's basically all there is to it! I like this solution primarily
because of the separation of concerns. The `ViewPager` has nothing to do
with the flipping back and forth of cards so no need to work through any
complex behaviour for switching the current fragment at the adapter
level.

If you want to take a look at the code I used to make this post, I've
put it up on [Github](https://github.com/jamesmccann/card-view-pager-example)

Cheers to [Benjamin Roesner](http://drbl.in/gGzR) for the Nexus template!











