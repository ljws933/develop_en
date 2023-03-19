---  
layout: post  
title: "Lifecycle changes when moving the screen with Navigation"  
categories: android
tags: android 
comments: true
showAds: true
description: When you use Jetpack's Navigation to move the screen, Activity, Fragment checks exactly when and how the life cycle circulates.
---

Logs are output for each lifecycle start.

### 1. Lifecycle changes when first executed

- By default, each override function is called first, and then the lifecycleScope for each state is called.
- onCreate -> onStart -> onResume

![https://blog.kakaocdn.net/dn/Zwso4/btrUumIeFXU/Rk5BJTjmNuCLpejbdrx31K/img.png](https://blog.kakaocdn.net/dn/Zwso4/btrUumIeFXU/Rk5BJTjmNuCLpejbdrx31K/img.png)

### 2. Lifecycle changes when confirming the currently running app screen (square button) and returning

- This does not affect the lifecycle.

### 3. Lifecycle changes when going to the Home screen and returning

- (Go to Home) -> onPause -> (Return) -> onStart -> onResume

![https://blog.kakaocdn.net/dn/EVMKJ/btrUunUICql/uULHLlSOy9Rj3NkcztC7yK/img.png](https://blog.kakaocdn.net/dn/EVMKJ/btrUunUICql/uULHLlSOy9Rj3NkcztC7yK/img.png)

### 4. Lifecycle changes when returning after viewing a completely different screen

- onPause -> onStart -> onResume

![https://blog.kakaocdn.net/dn/blHoHe/btrUthVuUCP/soHRXMByDz4Nc4NfX5p1qK/img.png](https://blog.kakaocdn.net/dn/blHoHe/btrUthVuUCP/soHRXMByDz4Nc4NfX5p1qK/img.png)

However, if each lifecycleScope is output only once with `lifecycleScope.launchWhenCreated { }` instead of specifying with `repeatOnLifecycle`, it will be output only once.

### 5. Conclusion

The lifecycle of Activity is output first, and then the lifecycle of lifecycleScope is output.

---

# Fragment

Check the lifecycle changes in Fragment when moving screens.

The main problem that occurs in Fragment is that the same data is set twice due to loading the data being observed again when returning from a different Fragment. Consider measures to prevent this by checking the lifecycle.

![https://blog.kakaocdn.net/dn/Mo0I0/btrUulWOIb5/8Hx0EFlHiANFBnUinvuVN0/img.png](https://blog.kakaocdn.net/dn/Mo0I0/btrUulWOIb5/8Hx0EFlHiANFBnUinvuVN0/img.png)

### Test process

1. Create two fragments, FirstFragment and SecondFragment.
2. Attach logs to FirstFragment to see the lifecycle.
3. Move from FirstFragment to SecondFragment and return to FirstFragment.
4. Check the lifecycle changes during the return process.
5. Use navigation to move between fragments.

[Full code of FirstFragment]

```
class FirstFragment : Fragment() {
    private lateinit var binding: FragmentFirstBinding
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentFirstBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        Timber.d("FirstFragment onViewCreated")
        binding.nextFragment.setOnClickListener {
            val action = FirstFragmentDirections.actionFirstFragmentToSecondFragment()
            findNavController().navigate(action)
        }
    }

	// Log output according to life cycle change of fragment
    override fun onStart() {
        super.onStart()
        Timber.d("FirstFragment onStart")
    }

    override fun onResume() {
        super.onResume()
        Timber.d("FirstFragment onResume")
    }

    override fun onPause() {
        super.onPause()
        Timber.d("FirstFragment onPause")
    }

    override fun onStop() {
        super.onStop()
        Timber.d("FirstFragment onStop")
    }

    override fun onDestroy() {
        super.onDestroy()
        Timber.d("FirstFragment onDestroy")
    }
}
```

### 1. Lifecycle changes when opening the first fragment

- onViewCreated -> onStart -> onResume

![https://blog.kakaocdn.net/dn/dAQyPd/btrUuX9r7Yh/MoqCNyYxCrXZ9tDkyYL290/img.png](https://blog.kakaocdn.net/dn/dAQyPd/btrUuX9r7Yh/MoqCNyYxCrXZ9tDkyYL290/img.png)

### 2. Lifecycle changes when using navigation to move from the first fragment to the second fragment

- When moving with the graph, the previous fragment remains in the stop state without being destroyed.
- onPause -> onStop

![https://blog.kakaocdn.net/dn/pHS4B/btrUwrB9rdP/Wv6c3orhmPrrPD3nUlxKi0/img.png](https://blog.kakaocdn.net/dn/pHS4B/btrUwrB9rdP/Wv6c3orhmPrrPD3nUlxKi0/img.png)

### 3. Lifecycle changes when using navigation to return from the second fragment to the first fragment

- Although it was not destroyed, it runs from onViewCreated again.
- onViewCreated -> onStart -> onResume

![https://blog.kakaocdn.net/dn/dm2DD8/btrUuBekoq8/HVeb3kW4Lq7KuKIKH5Zoy0/img.png](https://blog.kakaocdn.net/dn/dm2DD8/btrUuBekoq8/HVeb3kW4Lq7KuKIKH5Zoy0/img.png)

### 4. Lifecycle changes when completely closing the activity

- The Fragment is destroyed only when the Activity is completely closed.
- onPause -> onStop -> onDestroy

![https://blog.kakaocdn.net/dn/lpi9u/btrUt2XPJDN/Mz1fhIeZXQa7MCX0VlEkGk/img.png](https://blog.kakaocdn.net/dn/lpi9u/btrUt2XPJDN/Mz1fhIeZXQa7MCX0VlEkGk/img.png)

---

### Summary of each screen's lifecycle

- The lifecycle of Activity is output first, and then the lifecycle of lifecycleScope is output.
- In Fragment, the previous fragment remains in the stop state without being destroyed when moving with the graph, and it runs onViewCreated again when returning.