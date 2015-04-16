### Tips

To allow psql access in Linux without password<br/>
```
echo 'localhost:5432:*:egdclient:egdpassword' >> ~/.pgpass
```