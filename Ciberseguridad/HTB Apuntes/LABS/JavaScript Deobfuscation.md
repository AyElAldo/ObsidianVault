#lab 
## Try to study the HTML code of the webpage, and identify used JavaScript code within it. What is the name of the JavaScript file being used?

Inspeccionar >> Sources
## Once you find the JavaScript code, try to run it to see if it does any interesting functions. Did you get something in return?

https://jsconsole.com/ >> Pegar cpdigo
## As you may have noticed, the JavaScript code is obfuscated. Try applying the skills you learned in this module to deobfuscate the code, and retrieve the 'flag' variable.

https://matthewfl.com/unPacker.html

HTB{n'+'3v3r_'+'run_0'+'bfu5c'+'473d_'+'c0d3!'+'}
HTB{n3v3r_run_0bfu5c473d_c0d3!}
## Try to Analyze the deobfuscated JavaScript code, and understand its main functionality. Once you do, try to replicate what it's doing to get a secret key. What is the key?

```shell
curl http://154.57.164.68:32480/keys.php -X POST -d "keys=prueba" 
```

4150495f70336e5f37333537316e365f31355f66756e
## Once you have the secret key, try to decide it's encoding method, and decode it. Then send a 'POST' request to the same previous page with the decoded key as "key=DECODED_KEY". What is the flag you got?
```shell
curl -s http://154.57.164.68:32480/keys.php -X POST -d "keys=prueba" | xxd -p -r
```


