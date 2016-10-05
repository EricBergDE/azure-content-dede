<properties
	pageTitle="Azure Mobile Engagement iOS-SDK für die Reach-Integration | Microsoft Azure"
	description="Neueste Updates und Verfahren für das iOS-SDK für Azure Mobile Engagement"
	services="mobile-engagement"
	documentationCenter="mobile"
	authors="piyushjo"
	manager="erikre"
	editor="" />

<tags
	ms.service="mobile-engagement"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-ios"
	ms.devlang="objective-c"
	ms.topic="article"
	ms.date="09/14/2016"
	ms.author="piyushjo" />

#So integrieren Sie Engagement Reach auf iOS

Bevor Sie die in diesem Leitfaden beschriebenen Schritte ausführen, müssen Sie die im Dokument [Integrieren von Mobile Engagement unter iOS](mobile-engagement-ios-integrate-engagement.md) beschriebenen Verfahren ausführen.

Diese Dokumentation erfordert XCode 8. Wenn Sie auf XCode 7 tatsächlich nicht verzichten können, bietet sich das [iOS Engagement SDK 3.2.4](https://aka.ms/r6oouh) an. In dieser Vorgängerversion tritt bei Ausführung auf iOS 10-Geräten ein bekannter Fehler auf: Systembenachrichtigungen werden nicht umgesetzt. Zur Behebung dieses Problems müssen Sie die veraltete API `application:didReceiveRemoteNotification:` wie folgt in Ihrer App-Stellvertretung implementieren:

	- (void)application:(UIApplication*)application
	didReceiveRemoteNotification:(NSDictionary*)userInfo
	{
	    [[EngagementAgent shared] applicationDidReceiveRemoteNotification:userInfo fetchCompletionHandler:nil];
	}

> [AZURE.IMPORTANT] **Wir empfehlen diese Problemumgebung nicht**, weil diese iOS-API veraltet ist und sich dieses Verhalten in anstehenden (auch kleineren) iOS-Versionsupgrades ändern kann. Sie sollten so bald wie möglich zu XCode 8 wechseln.

### Aktivieren der App für den Empfang von stillen Pushbenachrichtigungen

[AZURE.INCLUDE [mobile-engagement-ios-silent-push](../../includes/mobile-engagement-ios-silent-push.md)]

##Schritte für die Integration

### Einbetten des Engagement Reach-SDK in Ihr iOS-Projekt

-   Fügen Sie das Reach-SDK in Ihr Xcode-Projekt ein. Wechseln Sie in Xcode zur **Project \\> Add to project**, und wählen Sie den Ordner `EngagementReach`.

### Ändern des Anwendungsdelegaten

-   Importieren Sie am Beginn Ihrer Implementierungsdatei das Engagement Reach-Modul:

		[...]
		#import "AEReachModule.h"

-   Erstellen Sie in der Methode `applicationDidFinishLaunching:` oder `application:didFinishLaunchingWithOptions:` ein Reach-Modul, und übergeben Sie es an Ihre vorhandene Zeile für die Engagement-Initialisierung:

		- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
		  AEReachModule* reach = [AEReachModule moduleWithNotificationIcon:[UIImage imageNamed:@"icon.png"]];
		  [EngagementAgent init:@"Endpoint={YOUR_APP_COLLECTION.DOMAIN};SdkKey={YOUR_SDK_KEY};AppId={YOUR_APPID}" modules:reach, nil];
		  [...]

		  return YES;
		}

-   Legen Sie die Zeichenfolge **'icon.png'** auf den Namen der Bilddatei fest, die Sie als Benachrichtigungssymbol verwenden möchten.
-   Wenn Sie die Option *Badgewert aktualisieren* in Reach-Kampagnen verwenden oder systemeigene Pushkampagnen – </SaaS/Reach API/Campaign format/Native Push> campaigns – einsetzen möchten, müssen Sie dem Reach-Modul die Verwaltung des Badgesymbols gestatten (Anwendungsbadge wird automatisch gelöscht und der von Engagement gespeicherte Wert wird bei jedem Start bzw. bei jeder Aktivierung der Anwendung zurückgesetzt). Dies geschieht durch Hinzufügen der folgenden Codezeile nach der Initialisierung des Reach-Moduls:

		[reach setAutoBadgeEnabled:YES];

-   Wenn Sie Pushvorgänge für Reach-Daten verarbeiten möchten, muss der Anwendungsdelegat mit dem `AEReachDataPushDelegate`-Protokoll konform sein. Fügen Sie nach der Initialisierung des Reach-Moduls die folgende Zeile ein:

		[reach setDataPushDelegate:self];

-   Anschließend können Sie die Methoden `onDataPushStringReceived:` und `onDataPushBase64ReceivedWithDecodedBody:andEncodedBody:` im Anwendungsdelegaten implementieren:

		-(BOOL)didReceiveStringDataPushWithCategory:(NSString*)category body:(NSString*)body
		{
		   NSLog(@"String data push message with category <%@> received: %@", category, body);
		   return YES;
		}

		-(BOOL)didReceiveBase64DataPushWithCategory:(NSString*)category decodedBody:(NSData *)decodedBody encodedBody:(NSString *)encodedBody
		{
		   NSLog(@"Base64 data push message with category <%@> received: %@", category, encodedBody);
		   // Do something useful with decodedBody like updating an image view
		   return YES;
		}

### Kategorie

Der category-Parameter ist optional, wenn Sie eine Datenpushkampagne erstellen und eine Filterung von Datenpushvorgängen ermöglichen. Dies ist nützlich, wenn Sie unterschiedliche Arten von `Base64`-Daten pushen und vor der Analyse deren Typ ermitteln möchten.

**Ihre Anwendung ist nun bereit, Reach-Inhalte zu empfangen und anzuzeigen!**

##Jederzeitiges Empfangen von Ankündigungen und Umfragen

Engagement kann über APNS (Apple Push Notification Service) jederzeit Reach-Benachrichtigungen an Ihre Endbenutzer senden.

Zur Aktivierung dieser Funktionalität müssen Sie Ihre Anwendung auf Apple-Pushbenachrichtigungen vorbereiten und Ihren Anwendungsdelegaten ändern.

### Vorbereiten Ihrer Anwendung auf Apple-Pushbenachrichtigungen

Folgen Sie den Anweisungen im Handbuch: [Vorbereiten der Anwendung für Apple-Pushbenachrichtigungen](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html#//apple_ref/doc/uid/TP40012582-CH26-SW6).

### Hinzufügen des erforderlichen Clientcodes

*Jetzt sollte Ihre Anwendung über ein registriertes Apple-Pushzertifikat im Engagement-Front-End verfügen.*

Sofern nicht bereits geschehen, müssen Sie Ihre Anwendung für den Empfang von Pushbenachrichtigungen registrieren. Fügen Sie die folgende Zeile beim Start der Anwendung ein (typischerweise in `application:didFinishLaunchingWithOptions:`):

	if ([application respondsToSelector:@selector(registerUserNotificationSettings:)]) {
	  	[application registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert) categories:nil]];
	  	[application registerForRemoteNotifications];
	}
	else {
	  	[application registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeAlert)];
	}

Anschließend müssen Sie für Engagement das von Apple-Servern zurückgegebene Gerätetoken bereitstellen. Dies wird mit der Methode `application:didRegisterForRemoteNotificationsWithDeviceToken:` in Ihrem Anwendungsdelegaten erreicht:

	- (void)application:(UIApplication*)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)deviceToken
	{
	    [[EngagementAgent shared] registerDeviceToken:deviceToken];
	}

Anschließend muss das Engagement-SDK darüber informiert werden, wenn Ihre Anwendung eine Remotebenachrichtigung erhält. Hierzu rufen Sie die Methode `applicationDidReceiveRemoteNotification:fetchCompletionHandler:` in Ihrem Anwendungsdelegaten auf:

	- (void)application:(UIApplication*)application didReceiveRemoteNotification:(NSDictionary*)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))handler
	{
		[[EngagementAgent shared] applicationDidReceiveRemoteNotification:userInfo fetchCompletionHandler:handler];
	}

> [AZURE.NOTE] In iOS 7 wird die oben genannte Methode eingeführt. Wenn Ihr Ziel iOS < 7 ist, implementieren Sie unbedingt die Methode `application:didReceiveRemoteNotification:` in Ihrem Anwendungsdelegaten, und rufen Sie `applicationDidReceiveRemoteNotification` auf dem EngagementAgent durch Übergeben von NULL anstelle des `handler`Arguments auf:

	- (void)application:(UIApplication*)application
	didReceiveRemoteNotification:(NSDictionary*)userInfo
	{
		[[EngagementAgent shared] applicationDidReceiveRemoteNotification:userInfo fetchCompletionHandler:nil];
	}

> [AZURE.IMPORTANT] Standardmäßig steuert Engagement Reach den completionHandler. Wenn Sie den Block `handler` in Ihrem Code manuell bearbeiten möchten, können Sie NULL für das Argument `handler`übergeben und den „completion“-Block selbst kontrollieren. Unter `UIBackgroundFetchResult`-Typ finden Sie eine Liste möglicher Werte.


### Vollständiges Beispiel

Hier sehen Sie ein vollständiges Beispiel für die Integration:

	#pragma mark -
	#pragma mark Application lifecycle

	- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions
	{
	  /* Reach module */
	  AEReachModule* reach = [AEReachModule moduleWithNotificationIcon:[UIImage imageNamed:@"icon.png"]];
	  [reach setAutoBadgeEnabled:YES];

	  /* Engagement initialization */
	  [EngagementAgent init:@"Endpoint={YOUR_APP_COLLECTION.DOMAIN};SdkKey={YOUR_SDK_KEY};AppId={YOUR_APPID}" modules:reach, nil];
	  [[EngagementAgent shared] setPushDelegate:self];

	  /* Views */
	  [window addSubview:[tabBarController view]];
	  [window makeKeyAndVisible];

	  [application registerForRemoteNotificationTypes:UIRemoteNotificationTypeAlert|UIRemoteNotificationTypeBadge|UIRemoteNotificationTypeSound];
	  return YES;
	}

	- (void)application:(UIApplication*)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)deviceToken
	{
	  [[EngagementAgent shared] registerDeviceToken:deviceToken];
	}

	- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))handler
	{
		[[EngagementAgent shared] applicationDidReceiveRemoteNotification:userInfo fetchCompletionHandler:handler];
	}

### Bei vorhandener eigener Implementierung von „UNUserNotificationCenterDelegate“

Das SDK hat auch eine eigene Implementierung des Protokolls „UNUserNotificationCenterDelegate“. Es wird vom SDK verwendet, um den Lebenszyklus der Engagement-Benachrichtigungen auf Geräten zu überwachen, die unter iOS 10 oder höher ausgeführt werden. Wenn das SDK Ihre Stellvertretung erkennt, verwendet es nicht seine eigene Implementierung, da es pro Anwendung nur eine „UNUserNotificationCenter“-Stellvertretung geben darf. Dies bedeutet, dass Sie Ihrer eigenen Stellvertretung die Engagement-Logik hinzufügen müssen.

Hierfür gibt es zwei Möglichkeiten.

Durch einfaches Weiterleiten der Stellvertretungsaufrufe an das SDK:

	#import <UIKit/UIKit.h>
	#import "EngagementAgent.h"
	#import <UserNotifications/UserNotifications.h>


	@interface MyAppDelegate : NSObject <UIApplicationDelegate, UNUserNotificationCenterDelegate>
	@end

	@implementation MyAppDelegate

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
	{
	  // Your own logic.

	  [[EngagementAgent shared] userNotificationCenterWillPresentNotification:notification withCompletionHandler:completionHandler]
	}

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler
	{
	  // Your own logic.

	  [[EngagementAgent shared] userNotificationCenterDidReceiveNotificationResponse:response withCompletionHandler:completionHandler]
	}
	@end

Oder durch Erben von der `AEUserNotificationHandler`-Klasse

	#import "AEUserNotificationHandler.h"
	#import "EngagementAgent.h"

	@interface CustomUserNotificationHandler :AEUserNotificationHandler
	@end

	@implementation CustomUserNotificationHandler

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
	{
	  // Your own logic.

	  [super userNotificationCenter:center willPresentNotification:notification withCompletionHandler:completionHandler];
	}

	- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse: UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler
	{
	  // Your own logic.

	  [super userNotificationCenter:center didReceiveNotificationResponse:response withCompletionHandler:completionHandler];
	}

	@end

> [AZURE.NOTE] Sie können bestimmen, ob eine Benachrichtigung von Engagement stammt oder nicht, indem das zugehörige `userInfo`-Wörterbuch an die `isEngagementPushPayload:`-Klassenmethode des Agents übergeben wird.

##Anpassen von Kampagnen

### Benachrichtigungen

Es gibt zwei Arten von Benachrichtigungen: System- und anwendungsinterne Benachrichtigungen.

Systembenachrichtigungen werden von iOS verarbeitet und können nicht angepasst werden.

Anwendungsinterne Benachrichtigungen bestehen aus einer Ansicht, die dynamisch in das aktuelle Anwendungsfenster eingefügt wird. Dies wird als Benachrichtigungsüberlagerung bezeichnet. Benachrichtigungsoverlays eignen sich besonders für eine schnelle Integration, da für sie keine Änderung von Ansichten in Ihrer Anwendung erforderlich ist.

#### Layout

Um das Layout Ihrer anwendungsinternen Benachrichtigungen zu ändern, können Sie einfach die Datei `AENotificationView.xib` an Ihre Anforderungen anpassen. Sie müssen hierbei jedoch darauf achten, dass die Tagwerte und -typen der vorhandenen Unteransichten beibehalten werden.

Standardmäßig werden anwendungsinterne Benachrichtigungen am unteren Bildschirmrand angezeigt. Wenn Sie eine Anzeige der Benachrichtigungen am oberen Bildschirmrand bevorzugen, bearbeiten Sie die bereitgestellte Datei `AENotificationView.xib`, und ändern Sie die Eigenschaft `AutoSizing` der Hauptansicht, sodass die Benachrichtigungen weiterhin im oberen Bereich der übergeordneten Ansicht angezeigt werden können.

#### Categories

Wenn Sie das bereitgestellte Layout ändern, ändern Sie das Aussehen aller Ihrer Benachrichtigungen. Mithilfe von Kategorien können Sie verschiedene zielgerichtete Layouts (Verhaltensweisen) für Benachrichtigungen definieren. Eine Kategorie kann beim Erstellen einer Reach-Kampagne angegeben werden. Bedenken Sie, dass Sie mithilfe von Kategorien auch Ankündigungen und Umfragen anpassen können (dies wird weiter unten in diesem Dokument beschrieben).

Um einen Kategorie-Handler für Ihre Benachrichtigungen zu registrieren, müssen Sie nach der Initialisierung des Reach-Moduls einen Aufruf hinzufügen.

	AEReachModule* reach = [AEReachModule moduleWithNotificationIcon:[UIImage imageNamed:@"icon.png"]];
	[reach registerNotifier:myNotifier forCategory:@"my_category"];
	...

Bei `myNotifier` muss es sich um eine Instanz eines Objekts handeln, das mit dem `AENotifier`-Protokoll konform ist.

Sie können die Protokollmethoden selbst implementieren oder erneut die vorhandene `AEDefaultNotifier`-Klasse implementieren, mit der bereits die meisten Aufgaben ausgeführt werden.

Wenn Sie beispielsweise die Benachrichtigungsansicht für eine spezifische Kategorie neu definieren möchten, können Sie diesem Beispiel folgen:

	#import "AEDefaultNotifier.h"
	#import "AENotificationView.h"
	@interface MyNotifier : AEDefaultNotifier
	@end

	@implementation MyNotifier

	-(NSString*)nibNameForCategory:(NSString*)category
	{
	  return "MyNotificationView";
	}

	@end

Dieses einfache Beispiel einer Kategorie setzt voraus, dass Sie in Ihrem Hauptanwendungspaket über eine Datei namens `MyNotificationView.xib` verfügen. Wenn die Methode keine entsprechende `.xib`-Datei findet, wird die Benachrichtigung nicht angezeigt und Engagement gibt eine entsprechende Meldung in der Konsole aus.

Die bereitgestellte NIB-Datei sollte den folgenden Regeln entsprechen:

-   Sie sollte nur eine Ansicht enthalten.
-   Unteransichten sollten dieselben Typen aufweisen wie diejenigen in der bereitgestellten NIB-Datei namens `AENotificationView.xib`
-   Unteransichten sollten dieselben Tags aufweisen wie diejenigen in der bereitgestellten NIB-Datei namens `AENotificationView.xib`

> [AZURE.TIP] Kopieren Sie einfach die bereitgestellte NIB-Datei namens `AENotificationView.xib`, und beginnen Sie Ihre Arbeit mit dieser Datei. Seien Sie jedoch vorsichtig, die Ansicht in dieser NIB-Datei ist mit der Klasse `AENotificationView` verknüpft. Diese Klasse definiert die Methode `layoutSubViews` neu, um die zugehörigen Unteransichten gemäß Kontext zu verschieben und in der Größe zu ändern. Sie können die Klasse durch `UIView` oder eine benutzerdefinierte Ansichtenklasse ersetzen.

Wenn Sie eine umfangreichere Anpassung der Benachrichtigungen benötigen (wenn Sie beispielsweise Ihre Ansicht direkt aus dem Code laden möchten), sollten Sie einen Blick auf den bereitgestellten Quellcode und die Klassendokumentation von `Protocol ReferencesDefaultNotifier` und `AENotifier` werfen.

Beachten Sie, dass Sie dieselben Notifier für mehrere Kategorien verwenden können.

Sie können auch den standardmäßigen Notifier folgendermaßen neu definieren:

	AEReachModule* reach = [AEReachModule moduleWithNotificationIcon:[UIImage imageNamed:@"icon.png"]];
	[reach registerNotifier:myNotifier forCategory:kAEReachDefaultCategory];

##### Benachrichtigungsverarbeitung

Bei Verwendung der Standardkategorie werden für das `AEReachContent`-Objekt einige Lebenszyklusmethoden aufgerufen, um Statistiken bereitzustellen und den Kampagnenstatus zu aktualisieren:

-   Bei Anzeige der Benachrichtigung in der Anwendung wird von die `displayNotification`-Methode von `AEReachModule` aufgerufen (diese liefert Statistiken), wenn `handleNotification:` den Wert `YES` zurückgibt.
-   Wird die Benachrichtigung geschlossen, wird die Methode `exitNotification` aufgerufen, es werden Statistiken bereitgestellt, und es können weitere Kampagnen verarbeitet werden.
-   Wird auf die Benachrichtigung geklickt, wird `actionNotification` aufgerufen, es werden Statistiken bereitgestellt, und die verknüpfte Aktion wird ausgeführt.

Wenn Ihre Implementierung von `AENotifier` das Standardverhalten umgeht, müssen Sie diese Lebenszyklusmethoden selbst aufrufen. Das folgende Beispiel zeigt einige Fälle, in denen das Standardverhalten umgangen wird:

-   Es erfolgt keine Erweiterung von `AEDefaultNotifier`, d. h. Sie haben die Kategorieverarbeitung von Grund auf implementiert.
-   Sie haben `prepareNotificationView:forContent:` außer Kraft gesetzt. Stellen Sie sicher, dass mindestens `onNotificationActioned` oder `onNotificationExited` einem Ihrer Benutzeroberflächen-Steuerelemente zugeordnet ist.

> [AZURE.WARNING] Wenn `handleNotification:` eine Ausnahme generiert, wird der Inhalt gelöscht, und es erfolgt ein Aufruf von `drop`. Dies wird in der Statistik berichtet, und es können weitere Kampagnen verarbeitet werden.

#### Einschließen von Benachrichtigungen als Bestandteil einer vorhandenen Ansicht

Overlays eignen sich hervorragend für eine schnelle Integration, sind aber gelegentlich nicht praktisch oder haben unerwünschte Nebeneffekte.

Wenn Sie mit dem Overlaysystem in einigen Ihrer Ansichten nicht zufrieden sind, können Sie es für diese Ansichten anpassen.

Sie können unser Benachrichtigungslayout in Ihre vorhandenen Ansichten einfügen. Die Implementierung kann auf zwei Arten erfolgen:

1.  Fügen Sie die Benachrichtigungsansicht mit dem Interface Builder hinzu.

	-   Öffnen Sie den *Interface Builder*
	-   Platzieren Sie ein `UIView`-Element der Größe 320 x 60 (oder 768 x 60 für das iPad) an der Stelle, an der die Benachrichtigung angezeigt werden soll.
	-   Legen Sie den Tagwert für diese Ansicht auf diesen Wert fest: **36822491**

2.  Fügen Sie die Benachrichtigungsansicht programmatisch hinzu. Fügen Sie einfach nach dem Initialisieren Ihrer Ansicht den folgenden Code ein:

		UIView* notificationView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 320, 60)]; //Replace x and y coordinate values to your needs.
		notificationView.tag = NOTIFICATION_AREA_VIEW_TAG;
		[self.view addSubview:notificationView];

Das Makro `NOTIFICATION_AREA_VIEW_TAG` befindet sich in `AEDefaultNotifier.h`.

> [AZURE.NOTE] Der standardmäßige Notifier erkennt automatisch, dass das Benachrichtigungslayout in dieser Ansicht enthalten ist und fügt kein Overlay hinzu.

### Ankündigungen und Umfragen

#### Layouts

Sie können die Dateien `AEDefaultAnnouncementView.xib` und `AEDefaultPollView.xib` ändern, solange Sie die Tagwerte und -typen der vorhandenen Unteransichten beibehalten.

#### Categories

##### Alternative Layouts

Wie Benachrichtigungen können die Kampagnenkategorien dazu verwendet werden, alternative Layouts für Ankündigungen und Umfragen bereitzustellen.

Um eine Kategorie für eine Ankündigung zu erstellen, müssen Sie **AEAnnouncementViewController** erweitern und nach der Initialisierung des Reach-Moduls registrieren:

	AEReachModule* reach = [AEReachModule moduleWithNotificationIcon:[UIImage imageNamed:@"icon.png"]];
	[reach registerAnnouncementController:[MyCustomAnnouncementViewController class] forCategory:@"my_category"];

> [AZURE.NOTE] Immer dann, wenn ein Benutzer auf eine Benachrichtigung für eine Ankündigung mit der Kategorie "my\_category" klickt, wird Ihr registrierter Ansichtencontroller (in diesem Fall `MyCustomAnnouncementViewController`) durch einen Aufruf der Methode `initWithAnnouncement:` initialisiert, und die Ansicht wird dem aktuellen Anwendungsfenster hinzugefügt.

In Ihrer Implementierung der Klasse `AEAnnouncementViewController` müssen Sie die Eigenschaft `announcement` lesen, um Ihre Unteransichten zu initialisieren. Siehe hierzu das nachfolgende Beispiel, in dem mithilfe der Eigenschaften `title` und `body` der Klasse `AEReachAnnouncement` zwei Labels initialisiert werden:

	-(void)loadView
	{
	    [super loadView];

	    UILabel* titleLabel = [[UILabel alloc] initWithFrame:CGRectMake(10, 20, 300, 60)];
	    titleLabel.font = [UIFont systemFontOfSize:32.0];
	    titleLabel.text = self.announcement.title;

	    UILabel* bodyLabel = [[UILabel alloc] initWithFrame:CGRectMake(10, 20, 300, 60)];
	    bodyLabel.font = [UIFont systemFontOfSize:24.0];
	    bodyLabel.text = self.announcement.body;

	    [self.view addSubview:titleLabel];
	    [self.view addSubview:bodyLabel];
	}

Wenn Sie Ihre Ansichten nicht selbst laden, sondern lediglich das Ansichtenlayout der Standardankündigung wiederverwenden möchten, können Sie einfach über Ihren benutzerdefinierten Ansichtencontroller die bereitgestellte Klasse `AEDefaultAnnouncementViewController` erweitern. In diesem Fall duplizieren Sie die NIB-Datei `AEDefaultAnnouncementView.xib`, und benennen Sie sie um, sodass Sie von Ihrem benutzerdefinierten Ansichtencontroller geladen werden kann (wenn Ihr Controller beispielsweise `CustomAnnouncementViewController` heißt, sollten Sie Ihrer NIB-Datei den Namen `CustomAnnouncementView.xib` geben).

Um die Standardkategorie für Ankündigungen zu ersetzen, registrieren Sie einfach Ihren benutzerdefinierten Ansichtencontroller für die in `kAEReachDefaultCategory` definierte Kategorie:

	[reach registerAnnouncementController:[MyCustomAnnouncementViewController class] forCategory:kAEReachDefaultCategory];

Umfragen können auf die gleiche Weise angepasst werden:

	AEReachModule* reach = [AEReachModule moduleWithNotificationIcon:[UIImage imageNamed:@"icon.png"]];
	[reach registerPollController:[MyCustomPollViewController class] forCategory:@"my_category"];

Dieses Mal muss der bereitgestellte `MyCustomPollViewController` zur Erweiterung von `AEPollViewController` verwendet werden. Alternativ können Sie eine Erweiterung über den Standardcontroller durchführen: `AEDefaultPollViewController`.

> [AZURE.IMPORTANT] Vergessen Sie nicht, entweder `action` (`submitAnswers:` für benutzerdefinierte Ansichtencontroller für Umfragen) oder die Methode `exit` aufzurufen, bevor der Ansichtencontroller verworfen wird. Andernfalls werden keine Statistiken gesendet (z. B. Analysen der Kampagne) und – noch wichtiger – weitere Kampagnen erhalten erst eine Benachrichtigung, wenn der Anwendungsprozess neu gestartet wurde.

##### Beispiel für die Implementierung

In dieser Implementierung wird die benutzerdefinierte Ankündigungsansicht aus einer externen XIB-Datei geladen.

Wie bei der erweiterten Benachrichtigungsanpassung wird empfohlen, sich den Quellcode der Standardimplementierung anzusehen.

`CustomAnnouncementViewController.h`

	//Interface
	@interface CustomAnnouncementViewController : AEAnnouncementViewController {
	  UILabel* titleLabel;
	  UITextView* descTextView;
	  UIWebView* htmlWebView;
	  UIButton* okButton;
	  UIButton* cancelButton;
	}

	@property (nonatomic, retain) IBOutlet UILabel* titleLabel;
	@property (nonatomic, retain) IBOutlet UITextView* descTextView;
	@property (nonatomic, retain) IBOutlet UIWebView* htmlWebView;
	@property (nonatomic, retain) IBOutlet UIButton* okButton;
	@property (nonatomic, retain) IBOutlet UIButton* cancelButton;

	-(IBAction)okButtonClicked:(id)sender;
	-(IBAction)cancelButtonClicked:(id)sender;

`CustomAnnouncementViewController.m`

	//Implementation
	@implementation CustomAnnouncementViewController
	@synthesize titleLabel;
	@synthesize descTextView;
	@synthesize htmlWebView;
	@synthesize okButton;
	@synthesize cancelButton;

	-(id)initWithAnnouncement:(AEReachAnnouncement*)anAnnouncement
	{
	  self = [super initWithNibName:@"CustomAnnouncementViewController" bundle:nil];
	  if (self != nil) {
	    self.announcement = anAnnouncement;
	  }
	  return self;
	}

	- (void) dealloc
	{
	  [titleLabel release];
	  [descTextView release];
	  [htmlWebView release];
	  [okButton release];
	  [cancelButton release];
	  [super dealloc];
	}

	- (void)viewDidLoad {
	  [super viewDidLoad];

	  /* Init announcement title */
	  titleLabel.text = self.announcement.title;

	  /* Init announcement body */
	  if(self.announcement.type == AEAnnouncementTypeHtml)
	  {
	    titleLabel.hidden = YES;
	    htmlWebView.hidden = NO;
	    [htmlWebView loadHTMLString:self.announcement.body baseURL:[NSURL URLWithString:@"http://localhost/"]];
	  }
	  else
	  {
	    titleLabel.hidden = NO;
	    htmlWebView.hidden = YES;
	    descTextView.text = self.announcement.body;
	  }

	  /* Set action button label */
	  if([self.announcement.actionLabel length] > 0)
	    [okButton setTitle:self.announcement.actionLabel forState:UIControlStateNormal];

	  /* Set exit button label */
	  if([self.announcement.exitLabel length] > 0)
	    [cancelButton setTitle:self.announcement.exitLabel forState:UIControlStateNormal];
	}

	#pragma mark Actions

	-(IBAction)okButtonClicked:(id)sender
	{
	    [self action];
	}

	-(IBAction)cancelButtonClicked:(id)sender
	{
	    [self exit];
	}

	@end

<!---HONumber=AcomDC_0921_2016-->