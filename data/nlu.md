## intent:greet
- hey
- hello
- hi
- good morning
- good evening
- hey there

## intent:goodbye
- bye
- goodbye
- see you around
- see you later

## intent:affirm
- yes
- indeed
- of course
- that sounds good
- correct

## intent:deny
- no
- never
- I don't think so
- don't like that
- no way
- not really

## intent:mood_great
- perfect
- very good
- great
- amazing
- wonderful
- I am feeling very good
- I am great
- I'm good

## intent:mood_unhappy
- sad
- very sad
- unhappy
- bad
- very bad
- awful
- terrible
- not very good
- extremely sad
- so sad

## intent:check_balance
- what is my balance <!-- no entity -->
- how much do I have on my [pink pig](source_account)
- how much do I have on my [savings](source_account) <!-- entity "source_account" has value "savings" -->
- how much do I have on my [savings account](source_account:savings) <!-- synonyms, method 1-->
- Could I pay in [yen](currency)?  <!-- entity matched by lookup table -->
- The U.S. zip code is [51000](zipcode)
- [87233](zipcode) is my home zip code

## synonym:savings   <!-- synonyms, method 2 -->
- pink pig

## regex:zipcode  <!-- My home zip code is 87666 注释掉正则表达式，当前样例无法被识别出实体。说明正则表达式的特征可以增强实体识别功能 -->
- [0-9]{5}

## lookup:currencies   <!-- lookup table list -->
- Yen
- USD
- Euro