---  
layout: post  
title: "Usage of registerForActivityResult & Reason of startActivityForResult is deprecated"  
categories: android
tags: android 
comments: true
showAds: true
description: The reason why startActivityForResult is Deprecated is that it is error prone, difficult to use correctly, and sometimes causes conflicts. Use registerForActivityResult to receive a callback when opening and closing Activity, and registerForActivityResult to open and retrieve picture files from the gallery.
---

### **Using registerForActivityResult and the reason for the deprecation of startActivityForResult**
1. Find out why startActivityForResult has been deprecated.
2. Open and close an Activity using registerForActivityResult and receive callbacks.
3. Open the gallery and receive a picture file using registerForActivityResult.

---

## **Reasons for the deprecation of startActivityForResult**

- Previously, startActivityForResult was defined within an Activity or Fragment.
    - As a result, if the new Activity used a lot of memory, there were cases where the previous Activity was killed and the callback was not received.
    - To prevent this, the part that runs the new Activity and the part that handles the callback were separated.
        - This is **registerForActivityResult**.

---

## **Implementing registerForActivityResult**

We will implement two cases:

1. Use registerForActivityResult to open and close an Activity, receive data and callbacks, and display the received data on the existing screen.
2. Use registerForActivityResult to open the gallery, receive the selected picture file, and display it on the existing screen.

---

### **Using registerForActivityResult to open an Activity**

The basic layout is as follows:

- FirstFragment.kt
    - Click the Button in the Fragment to open an Activity.
    - Display the data received from the callback in a TextView.

<img src="https://blog.kakaocdn.net/dn/WrVXr/btrVBoj3gzX/dOncW5zIZRAPZklgVhBKN0/img.png" width="350" alt="Usage of registerForActivityResult">

- **FirstFragment.kt**

``` kotlin
// registerForActivityResult must be declared and defined in the global section of FirstFragment.kt.private var resultLauncher =
	registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
// Print the received String data when it is not null or blank.
            when {
                !result.data?.getStringExtra("data").isNullOrBlank() -> {
                    binding.textView.text = result.data?.getStringExtra("data")
                    binding.textView2.text = "Returned from TestActivity"
                }
}

// Define in onViewCreated
binding.openActivity.setOnSingleClickListener {
    val intent = Intent(context, FirstActivity::class.java)
// Specify which Intent to open here.// Could be a general Activity or an image Intent.
    resultLauncher.launch(intent)
}

```

- **FirstAcitivty.kt**
    - An Activity opened using registerForActivityResult.
    - When the Close button is clicked, send data with the callback to the previous screen.

<img src="https://blog.kakaocdn.net/dn/cbIW8B/btrVAurxbbD/rqCLqH91X8ZGUOyX8ZGUOy8hYIGVk/img.png" width="350" alt="">

``` kotlin
// ActivityResultLauncher must be declared and defined in the global section.private val resultLauncher =
    registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
// Print the received data in the log.
            Timber.d(result.data?.getStringExtra("data"))
        }
    }

// onCreate
binding.button.setOnSingleClickListener {
// Send data when exiting the Activity.
    intent.putExtra("data", "Data sent from FirstActivity")
    setResult(RESULT_OK, intent)

    finish()
}

```

- When you click the Close button in FirstActivity and return to FirstFragment, the result shows that the callback is received and the received data is displayed on the screen.

<img src="https://blog.kakaocdn.net/dn/czm8dG/btrVzlCn6Zy/15898tyQZszXWjK9a9m6UK/img.png" width="350" alt="Usage of registerForActivityResult">

---

### **Using registerForActivityResult to open the gallery and receive a picture file**

- FirstFragment.kt

``` kotlin
// The following variable is a global variable.private var bookImageBitmap: Bitmap? = null

// ActivityResultLauncher must be declared and defined outside.private var resultLauncher =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
            if (result.resultCode == Activity.RESULT_OK) {
// Print the received String data when it is not null or blank.
                when {
                    !result.data?.getStringExtra("data").isNullOrBlank() -> {
                        binding.textView.text = result.data?.getStringExtra("data")
                        binding.textView2.text = "Returned from TestActivity"
                    }
// Check if there is data received after returning from the gallery.
                    result.data?.data != null -> {
                        result.data?.data?.let { uri ->
                            if (result.resultCode == AppCompatActivity.RESULT_OK) {
// Convert the picture data to a Bitmap.
                                bookImageBitmap =
                                    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
                                        ImageDecoder.decodeBitmap(
                                            ImageDecoder.createSource(
                                                requireContext().contentResolver, uri
                                            )
                                        )
                                    } else {
                                        MediaStore.Images.Media.getBitmap(
                                            requireContext().contentResolver, uri
                                        )
                                    }
                                Glide.with(requireContext()).load(uri)
                                    .placeholder(R.drawable.ic_bottom_menu_search)
                                    .into(binding.imageView)
                            }
                        }
                    }
                }
            }
        }

// Define in onViewCreated
binding.openImageActivity.setOnSingleClickListener {
// Define an Intent to open the gallery.
    val intent = Intent(Intent.ACTION_GET_CONTENT)
    intent.type = "image/*"
    intent.flags = Intent.FLAG_ACTIVITY_SINGLE_TOP or Intent.FLAG_ACTIVITY_NO_HISTORY

    resultLauncher.launch(intent)
}

```

- When you click on openImageActivity to open the gallery:

<img src="https://blog.kakaocdn.net/dn/dhPMGf/btrVBd30Tbb/t9f1ovpaSG0feTGUUbTgF1/img.png" width="350" alt="Usage of registerForActivityResult">

---

### **Using registerForActivityResult and the reason for the deprecation of startActivityForResult**

1. Find out why startActivityForResult has been deprecated.
2. Open and close an Activity using registerForActivityResult and receive callbacks.
3. Open the gallery and receive a picture file using registerForActivityResult.