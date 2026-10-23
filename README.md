# mashninski-site-naviny

Бот, які збірае навіны пра штучны інтэлект, пераказвае іх па-беларуску і прапануе
гатовыя карткі ў раздзел [«ШІ-навіны»](https://mashninski.com/ai-naviny) на сайце
[mashninski.com](https://mashninski.com).

Працуе сам, па раскладзе ў GitHub Actions. Чалавек толькі правярае прапанаваныя карткі
і зацвярджае публікацыю.

## Як гэта працуе

```
стужкі і sitemap  →  collect.py  →  pipeline.py  →  publish.py  →  pull request у сайт
  16 крыніц          збор,          адбор, пераказ,   карткі        рэўю і зліццё =
                     дублі          праверка мовы     ў JSON        публікацыя
```

1. **Збор** (`collect.py`). Чытае RSS/Atom-стужкі OpenAI (навіны і alignment.openai.com),
   Google DeepMind, Hugging Face, Mistral, TechCrunch, VentureBeat, MIT Technology Review,
   Ars Technica, The Verge, Axios, 404 Media, South China Morning Post, Simon Willison
   і sitemap Anthropic і claude.com. Прыводзіць запісы да агульнага выгляду, адкідае ўжо
   бачанае, спонсарскія тэксты і старонкі-рубрыкі і склейвае адну навіну з розных крыніц
   у адну. Збой адной крыніцы не спыняе астатнія.
2. **Пайплайн** (`pipeline.py`). Адбор важнага — Claude Haiku, пераказ па-беларуску —
   Claude Sonnet. Далей лінтэр шукае русізмы і калькі (`lint.py`), пазначаныя сказы
   ідуць на выпраўленне. Усе выклікі мадэляў — праз Batch API, з месячным бюджэтам
   і запабежнікамі. Па жаданні тэкст канвертуецца ў тарашкевіцу праз
   [taraskevizer](https://github.com/GooseOb/taraskevizer) (`tarask.py`).
3. **Публікацыя** (`publish.py`). Гатовыя карткі адкрываюцца адным pull request’ам
   у рэпазіторый сайта, па файле на картку. Рэўю — у дыфе, зліццё — публікацыя.

Стан (што ўжо бачылі, што чакае генерацыі) захоўваецца ў SQLite-файле. Паміж
прагонамі Actions ён жыве ў кэшы, у git не трапляе.

## Стэк

Python 3.12 · feedparser · Beautiful Soup · Anthropic API (Batch) · SQLite ·
GitHub Actions · Node.js для канвертара ў тарашкевіцу.

## Запуск

```bash
pip install -r requirements.txt
python collect.py --dry-run   # паказаць новыя матэрыялы, нічога не захоўваючы
python -m pytest              # тэсты, сетка і ключы не патрэбныя
```

Пайплайну патрэбныя ключ Anthropic API ў зменнай `AI_NEWS_ANTHROPIC_KEY` і побач
рэпазіторый сайта: адтуль бяруцца стайлгайд, слоўнік тэрмінаў і напісанне імёнаў.
Публікацыі патрэбны токен GitHub з доступам да рэпазіторыя сайта (`AI_NEWS_SITE_TOKEN`).
Ключы захоўваюцца толькі ў GitHub Secrets і лакальным `.env.local`, у код не трапляюць.

## Раклад

Workflow `.github/workflows/harvest.yml` запускае Cloudflare Worker з суседняга
рэпазіторыя [mashninski-site-strymy](https://github.com/mashninski/mashninski-site-strymy)
раз на дзве гадзіны. Уласны `schedule` GitHub — запасны.
