# tbot
test bot
Coding Session Telebot - фреймворк для написання власних ботів для телеграм бота
Продовжуємо використання репозиторію, наповненому в попередній практичній роботі.
Програми на GO складаються з пакетів.
Зазвичай пакети залежать від інших зовнішніх або вбудованих в стандартну бібліотеку пакетів.
Щоб використовувати пакет потрібно спочатку його імпортувати. Це робиться за допомогою конструкції, що називається декларацією імпорту, що складається з ідентифікатору telebot який буде далі використовуватись в коді, та шляху до пакету що імпортується gopkg.in/telebot.v3:
import (
	"fmt"

	"github.com/spf13/cobra"
	telebot "gopkg.in/telebot.v3"
)
В даному випадку сервіс gopkg.in надає версіоновані url-адреси, які містять відповідні метадані для переправлення інструменту GO на чітко визначені репозиторії github
Задекларуємо змінну TeleToken, значення якої буде отримуватись автоматично із змінної середовища під час старту програми за допомогою функції os.Getenv:
var (
	// TeleToken bot
	TeleToken = os.Getenv("TELE_TOKEN")
)
Додамо код функції Run в блоці хендлера kbot cmd, це буде форматований вивід версії нашої програми при запуску та блок ініціалізації kbot, що являє собою створення нового боту з параметрами:
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Printf("kbot %s started", appVersion)
		kbot, err := telebot.NewBot(telebot.Settings{
			URL:	"",
			Token:	TeleToken,
			Poller: &telebot.LongPoller{Timeout: 10 * time.Second},
		})
	},
Додамо в цей же блок обробник помилок, швидше за все це буде помилка відсутності токена у змінних середовища, отже так її ї опишемо:
		if err !=nil {
			log.Fatalf("Please check TELE_TOKEN env variable. %s", err)
			return
		}
Код хендлера буде обробляти подію telebot.OnText, тоб-то коли ми будемо отримувати нові повідомлення в Телеграмі буде виконуватись функція, що містить параметр telebot.Context для подальшої його обробки. В кінці блоку додамо kbot.Start() для запуску хендлера та його код.
		kbot.Handle(telebot.OnText, func(m telebot.Context) error {
			log.Print(m.Message(),Payload, m.Text())
			return err
		})
		
		kbot.Start()
Компілюємо код. А перед тим зробимо форматування та вирівнювання коду, а також завантажимо пакети та залежності командою go get:
$ gofmt -s -w ./
$ go get
go: downloading gopkg.in/telebot.v3 v3.1.4
go: added gopkg.in/telebot.v3 v3.1.4
$ go build -ldflags "-X="github.com/vit-um/kbot/cmd.appVersion=v1.0.1
# github.com/vit-um/kbot/cmd
cmd/kbot.go:33:39: undefined: telebot.settings
cmd/kbot.go:40:8: undefined: log.FatalI
cmd/kbot.go:45:27: undefined: Payload

# виправляємо помилки в коді та повторюємо команди
$ gofmt -s -w ./                                                     
$ go build -ldflags "-X="github.com/vit-um/kbot/cmd.appVersion=v1.0.1
$ ./kbot
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  kbot [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  kbot        A brief description of your command
  version     A brief description of your command
Виходить що для запуску основного функціоналу програми потрібно два рази набирати її назву, не кошерно... Змінимо назву команди за допомогою введення аліасу:
var kbotCmd = &cobra.Command{
	Use:     "kbot",
	Aliases: []string{"go"},
	Short:   "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
Компілюємо код та запускаємо ще раз:
$ go build -ldflags "-X="github.com/vit-um/kbot/cmd.appVersion=v1.0.1
$ ./kbot go
kbot v1.0.1 started2023/11/12 15:16:38 Please check TELE_TOKEN env variable. telegram: Not Found (404)
Підготуємо бота. Відкриємо телеграм, знайдемо BotFather та створимо нового бота командою /newbot:
