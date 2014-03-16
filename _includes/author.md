<div class="user">
	<img class="userpic" src="{{include.author.avatar | xml_escape | uri_escape }}">
	{{include.title}}: <a href="{{include.author.site | xml_escape | uri_escape }}">{{include.author.name}}</a><br>
	GitHub: <a href="https://github.com/{{include.author.github | xml_escape | uri_escape }}">{{include.author.github}}</a><br>
	Twitter: <a href="https://twitter.com/{{include.author.twitter | xml_escape | uri_escape }}">@{{include.author.twitter}}</a>
</div>
