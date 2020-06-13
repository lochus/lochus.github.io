---
title: "HackPack: Super Secret Vault"
categories:
  - HackPack
tags:
  - HackPack
---


Downloading the index.php file for this challenge, we see the following code:

```
        <?php
      // this is how I store hashes right?
      $hash = "0e770334890835629000008642775106";
      if(array_key_exists("combination",$_REQUEST) && $_REQUEST["combination"] !== ''){
          //Isn't it great that == works in every language?
          if(array_key_exists("debug", $_REQUEST)){
              echo "<br> ". md5($_REQUEST["combination"]);
          }
          if(md5($_REQUEST["combination"]) == $hash){
              echo "<br> The Flag is flag{...}<br>";
          }
          else{
              echo "<br>Wrong!<br>";
          }

      }
?>

```

It looks like it takes a GET request for ?combination= in the URL, takes the md5 hash of the supplied input, and compares it to the hash above. This hash isn't easily crackable (with rockyou.txt and hashes.org), so it probably isn't a guessing challenge. However, we can use PHP Type Juggling to bypass the password check, as the password "240610708"'s md5 hash evaluates to 0 which, when compared with the hash, evaluates to true!

We get the flag:

**flag{!5_Ph9_5TronGly_7yPed?}**
