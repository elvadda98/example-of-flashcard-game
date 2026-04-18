from pathlib import Path
import re, json, textwrap
path = Path('output/vocab-flashcards.slides.html')
html = path.read_text(encoding='utf-8')
# Rebuild with more natural example sentences and emoji+Italian on relevant fronts.
# Simple targeted replacements for examples and some modes.
repls = {
"{en:'flight', it:'volo', mode:'mix', emoji:'✈️', note:'trip by plane', ex:'She booked a flight to Rome.'}":"{en:'flight', it:'volo', mode:'mix', emoji:'✈️', note:'trip by plane', ex:'We caught a flight to Milan at 6 a.m.'}",
"{en:'from', it:'da', mode:'it', emoji:'📍', note:'starting point'}":"{en:'from', it:'da', mode:'mix', emoji:'📍', note:'starting point', ex:'I’m from Bari, so I know the city well.'}",
"{en:'alone', it:'da solo', mode:'mix', emoji:'🧍', note:'without other people', ex:'He was alone at home after dinner.'}":"{en:'alone', it:'da solo', mode:'mix', emoji:'🧍', note:'without other people', ex:'She prefers to travel alone.'}",
"{en:'away', it:'di distanza', mode:'it', emoji:'📏', note:'as in Milan is one hour away'}":"{en:'away', it:'di distanza', mode:'mix', emoji:'📏', note:'as in Milan is one hour away', ex:'The hotel is only ten minutes away.'}",
"{en:'so', it:'quindi', mode:'it', emoji:'➡️', note:'result or consequence'}":"{en:'so', it:'quindi', mode:'mix', emoji:'➡️', note:'result or consequence', ex:'It was raining, so we stayed indoors.'}",
"{en:'took', it:'ha preso / presi', mode:'it', emoji:'🫳', note:'past of take'}":"{en:'took', it:'ha preso', mode:'mix', emoji:'🫳', note:'past of take', ex:'She took the train to work yesterday.'}",
"{en:'opposite', it:'opposto / di fronte', mode:'mix', emoji:'↔️', note:'other side or contrary position'}":"{en:'opposite', it:'opposto / di fronte', mode:'mix', emoji:'↔️', note:'other side or contrary position', ex:'The café is opposite the station.'}",
"{en:'heavy', it:'pesante', mode:'mix', emoji:'🏋️', note:'not light', ex:'This bag is too heavy for me.'}":"{en:'heavy', it:'pesante', mode:'mix', emoji:'🏋️', note:'not light', ex:'My backpack is too heavy today.'}",
"{en:'tired', it:'stanco', mode:'mix', emoji:'🥱', note:'needing rest', ex:'I feel tired after the lesson.'}":"{en:'tired', it:'stanco', mode:'mix', emoji:'🥱', note:'needing rest', ex:'I’m tired because I got home late.'}",
"{en:'also', it:'anche', mode:'it', emoji:'➕', note:'in addition'}":"{en:'also', it:'anche', mode:'mix', emoji:'➕', note:'in addition', ex:'She speaks French and also Spanish.'}",
"{en:'bored', it:'annoiato', mode:'mix', emoji:'😒', note:'not interested', ex:'She was bored during the long meeting.'}":"{en:'bored', it:'annoiato', mode:'mix', emoji:'😒', note:'not interested', ex:'I get bored if the lesson is too slow.'}",
"{en:'always', it:'sempre', mode:'mix', emoji:'♾️', note:'every time', ex:'He always arrives early.'}":"{en:'always', it:'sempre', mode:'mix', emoji:'♾️', note:'every time', ex:'She always drinks coffee before class.'}",
"{en:'answer', it:'risposta / rispondere', mode:'mix', emoji:'💬', note:'reply to a question', ex:'Please answer the question carefully.'}":"{en:'answer', it:'risposta / rispondere', mode:'mix', emoji:'💬', note:'reply to a question', ex:'Can you answer the phone for me?'}",
"{en:'found', it:'ha trovato / trovai', mode:'it', emoji:'🔎', note:'past of find'}":"{en:'found', it:'ha trovato', mode:'mix', emoji:'🔎', note:'past of find', ex:'I found my keys in the kitchen.'}",
"{en:'a little', it:'un po\'', mode:'it', emoji:'🤏', note:'small amount'}":"{en:'a little', it:'un po\'', mode:'mix', emoji:'🤏', note:'small amount', ex:'I’m a little tired today.'}",
"{en:'mistakes', it:'errori', mode:'mix', emoji:'❌', note:'things done incorrectly', ex:'Everyone makes mistakes when learning.'}":"{en:'mistakes', it:'errori', mode:'mix', emoji:'❌', note:'things done incorrectly', ex:'Mistakes are part of learning.'}",
"{en:'weather', it:'tempo', mode:'mix', emoji:'🌦️', note:'sun, rain, wind, clouds', ex:'The weather is perfect today.'}":"{en:'weather', it:'tempo', mode:'mix', emoji:'🌦️', note:'sun, rain, wind, clouds', ex:'The weather has changed a lot today.'}",
"{en:'hopefully', it:'si spera / speriamo', mode:'it', emoji:'🤞', note:'expressing hope'}":"{en:'hopefully', it:'si spera', mode:'mix', emoji:'🤞', note:'expressing hope', ex:'Hopefully, the train won’t be late.'}",
"{en:'square', it:'piazza', mode:'mix', emoji:'🏛️', note:'town square', ex:'We met in the square after lunch.'}":"{en:'square', it:'piazza', mode:'mix', emoji:'🏛️', note:'town square', ex:'Let’s meet in the square at 8.'}",
"{en:'valet', it:'parcheggiatore', mode:'it', emoji:'🅿️', note:'person who parks your car'}":"{en:'valet', it:'parcheggiatore', mode:'mix', emoji:'🅿️', note:'person who parks your car', ex:'The valet parked the car for us.'}",
"{en:'saving up', it:'risparmiando', mode:'mix', emoji:'💰', note:'putting money aside'}":"{en:'saving up', it:'risparmiando', mode:'mix', emoji:'💰', note:'putting money aside', ex:'I’m saving up for a new laptop.'}",
"{en:'how far', it:'quanto dista', mode:'it', emoji:'📐', note:'asking distance'}":"{en:'how far', it:'quanto dista', mode:'mix', emoji:'📐', note:'asking distance', ex:'How far is the station from here?'}",
"{en:'castle', it:'castello', mode:'mix', emoji:'🏰', note:'large old fortified building', ex:'The castle is on top of the hill.'}":"{en:'castle', it:'castello', mode:'mix', emoji:'🏰', note:'large old fortified building', ex:'We visited a castle near the sea.'}",
"{en:'village', it:'villaggio / paese', mode:'mix', emoji:'🏘️', note:'small town', ex:'She grew up in a small village.'}":"{en:'village', it:'villaggio / paese', mode:'mix', emoji:'🏘️', note:'small town', ex:'My grandparents live in a quiet village.'}",
"{en:'uphill', it:'in salita', mode:'emoji', emoji:'⛰️', note:'going upward'}":"{en:'uphill', it:'in salita', mode:'mix', emoji:'⛰️', note:'going upward', ex:'The road gets uphill after the bridge.'}",
"{en:'enough', it:'abbastanza', mode:'it', emoji:'✅', note:'as much as needed'}":"{en:'enough', it:'abbastanza', mode:'mix', emoji:'✅', note:'as much as needed', ex:'We have enough time for one more question.'}",
"{en:'by', it:'in / con', mode:'it', emoji:'🚆', note:'as in by train, by car, by plane'}":"{en:'by', it:'in / con', mode:'mix', emoji:'🚆', note:'as in by train, by car, by plane', ex:'We went there by train.'}",
"{en:'code', it:'codice', mode:'mix', emoji:'🔐', note:'number or set of symbols', ex:'I forgot the code for the door.'}":"{en:'code', it:'codice', mode:'mix', emoji:'🔐', note:'number or set of symbols', ex:'Enter the code to unlock the app.'}",
"{en:'subscription', it:'abbonamento', mode:'mix', emoji:'🧾', note:'regular paid access', ex:'My subscription expires next month.'}":"{en:'subscription', it:'abbonamento', mode:'mix', emoji:'🧾', note:'regular paid access', ex:'I renewed my subscription yesterday.'}",
"{en:'once', it:'una volta', mode:'it', emoji:'1️⃣', note:'one time'}":"{en:'once', it:'una volta', mode:'mix', emoji:'1️⃣', note:'one time', ex:'I’ve been there once before.'}",
"{en:'many', it:'molti', mode:'mix', emoji:'🔢', note:'a large number', ex:'There are many students in the class.'}":"{en:'many', it:'molti', mode:'mix', emoji:'🔢', note:'a large number', ex:'Many people prefer tea to coffee.'}",
"{en:'like', it:'come', mode:'it', emoji:'🪞', note:'as in like Milan, like me'}":"{en:'like', it:'come', mode:'mix', emoji:'🪞', note:'as in like Milan, like me', ex:'A city like Milan is always busy.'}",
"{en:'send', it:'mandare / inviare', mode:'mix', emoji:'📤', note:'to move to another person', ex:'Please send me the document.'}":"{en:'send', it:'mandare / inviare', mode:'mix', emoji:'📤', note:'to move to another person', ex:'I’ll send you the file tonight.'}",
"{en:'kind', it:'gentile', mode:'mix', emoji:'😊', note:'nice and caring', ex:'She is always kind to her students.'}":"{en:'kind', it:'gentile', mode:'mix', emoji:'😊', note:'nice and caring', ex:'That was really kind of you.'}",
"{en:'share', it:'condividere', mode:'mix', emoji:'🤝', note:'give part to others', ex:'Let’s share this idea with the group.'}":"{en:'share', it:'condividere', mode:'mix', emoji:'🤝', note:'give part to others', ex:'Could you share your notes with me?'}",
"{en:'job interview', it:'colloquio di lavoro', mode:'mix', emoji:'🧑‍💼', note:'meeting for a possible job'}":"{en:'job interview', it:'colloquio di lavoro', mode:'mix', emoji:'🧑‍💼', note:'meeting for a possible job', ex:'I have a job interview tomorrow morning.'}",
"{en:'driver', it:'autista / guidatore', mode:'mix', emoji:'🚕', note:'person who drives', ex:'The driver was very polite.'}":"{en:'driver', it:'autista / guidatore', mode:'mix', emoji:'🚕', note:'person who drives', ex:'The driver took us straight to the hotel.'}",
"{en:'recently', it:'recentemente', mode:'mix', emoji:'🕒', note:'not long ago', ex:'We recently moved to a new house.'}":"{en:'recently', it:'recentemente', mode:'mix', emoji:'🕒', note:'not long ago', ex:'I recently started a new course.'}",
"{en:'cuisine', it:'cucina', mode:'mix', emoji:'🍝', note:'food style of a country'}":"{en:'cuisine', it:'cucina', mode:'mix', emoji:'🍝', note:'food style of a country', ex:'Italian cuisine is famous all over the world.'}",
"{en:'stay', it:'restare / soggiornare', mode:'mix', emoji:'🛏️', note:'remain or spend time in a place', ex:'We will stay in Bari for two days.'}":"{en:'stay', it:'restare / soggiornare', mode:'mix', emoji:'🛏️', note:'remain or spend time in a place', ex:'We stayed at a small hotel near the beach.'}",
"{en:'uses', it:'utilizza', mode:'it', emoji:'🛠️', note:'third person singular of use'}":"{en:'uses', it:'utilizza', mode:'mix', emoji:'🛠️', note:'third person singular of use', ex:'She uses this app every day.'}",
"{en:'cloudy', it:'nuvoloso', mode:'mix', emoji:'☁️', note:'full of clouds', ex:'It looks cloudy this morning.'}":"{en:'cloudy', it:'nuvoloso', mode:'mix', emoji:'☁️', note:'full of clouds', ex:'It looks cloudy, so take an umbrella.'}",
"{en:'between', it:'tra / fra', mode:'it', emoji:'↔️', note:'in the middle of two things'}":"{en:'between', it:'tra / fra', mode:'mix', emoji:'↔️', note:'in the middle of two things', ex:'The bank is between the café and the pharmacy.'}",
"{en:'allow', it:'permettere', mode:'mix', emoji:'✅', note:'to let someone do something', ex:'The teacher allows questions at the end.'}":"{en:'allow', it:'permettere', mode:'mix', emoji:'✅', note:'to let someone do something', ex:'My phone doesn’t allow downloads here.'}",
"{en:'neighbor', it:'vicino di casa', mode:'mix', emoji:'🏡', note:'person living next door', ex:'My neighbor is very friendly.'}":"{en:'neighbor', it:'vicino di casa', mode:'mix', emoji:'🏡', note:'person living next door', ex:'Our neighbor helped us carry the bags.'}",
"{en:'far away', it:'lontano', mode:'it', emoji:'🌍', note:'at a great distance'}":"{en:'far away', it:'lontano', mode:'mix', emoji:'🌍', note:'at a great distance', ex:'The mountains look far away from here.'}",
"{en:'come to mind', it:'venire in mente', mode:'it', emoji:'💡', note:'appear in your thoughts'}":"{en:'come to mind', it:'venire in mente', mode:'mix', emoji:'💡', note:'appear in your thoughts', ex:'No good idea comes to mind right now.'}",
"{en:'take a flight', it:'prendere un volo', mode:'mix', emoji:'🛫', note:'travel by plane'}":"{en:'take a flight', it:'prendere un volo', mode:'mix', emoji:'🛫', note:'travel by plane', ex:'We need to take a flight early tomorrow.'}",
"{en:'shifts', it:'turni', mode:'mix', emoji:'🗓️', note:'work periods', ex:'She works night shifts this week.'}":"{en:'shifts', it:'turni', mode:'mix', emoji:'🗓️', note:'work periods', ex:'He’s doing night shifts all week.'}",
"{en:'patience', it:'pazienza', mode:'mix', emoji:'🧘', note:'ability to wait calmly', ex:'Teaching needs a lot of patience.'}":"{en:'patience', it:'pazienza', mode:'mix', emoji:'🧘', note:'ability to wait calmly', ex:'You need patience when learning a language.'}",
"{en:'outgoing', it:'socievole', mode:'mix', emoji:'🎉', note:'friendly and sociable', ex:'He is outgoing and easy to talk to.'}":"{en:'outgoing', it:'socievole', mode:'mix', emoji:'🎉', note:'friendly and sociable', ex:'She’s outgoing and makes friends easily.'}"
}
for old,new in repls.items():
    html = html.replace(old,new)
path.write_text(html, encoding='utf-8')
print('rewritten')
