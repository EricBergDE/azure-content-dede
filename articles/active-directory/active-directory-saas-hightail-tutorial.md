<properties
	pageTitle="Tutorial: Azure Active Directory-Integration mit Hightail | Microsoft Azure"
	description="Erfahren Sie, wie Sie das einmalige Anmelden für Azure Active Directory und Hightail konfigurieren."
	services="active-directory"
	documentationCenter=""
	authors="jeevansd"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/11/2016"
	ms.author="jeedes"/>


# Tutorial: Azure Active Directory-Integration in Hightail

Dieses Tutorial soll Ihnen zeigen, wie Sie Hightail in Azure Active Directory (Azure AD) integrieren können.

Die Integration von Hightail in Azure AD bietet die folgenden Vorteile:

- Sie können in Azure AD steuern, wer Zugriff auf Hightail hat
- Sie können es Benutzern ermöglichen, sich mit ihren Azure AD-Konten automatisch bei Hightail anzumelden (einmaliges Anmelden).
- Sie können Ihre Konten an einem zentralen Ort verwalten – dem Azure Active Directory-Portal

Weitere Informationen zur Integration von SaaS-Apps in Azure AD finden Sie unter [Was bedeuten Anwendungszugriff und einmaliges Anmelden mit Azure Active Directory?](active-directory-appssoaccess-whatis.md).

## Voraussetzungen

Um die Azure AD-Integration mit Hightail konfigurieren zu können, benötigen Sie Folgendes:

- Ein Azure AD-Abonnement
- Ein Hightail-Abonnement, für das einmaliges Anmelden aktiviert ist


> [AZURE.NOTE] Um die Schritte in diesem Tutorial zu testen, wird empfohlen, keine Produktionsumgebung zu verwenden.


Um die Schritte in diesem Tutorial zu testen, sollten Sie folgende Empfehlungen beachten:

- Sie sollten keine Produktionsumgebung verwenden, sofern dies nicht erforderlich ist.
- Wenn Sie keine Azure AD-Testumgebung haben, können Sie [hier](https://azure.microsoft.com/pricing/free-trial/) eine einmonatige Testversion anfordern.


## Beschreibung des Szenarios
Ziel dieses Tutorials ist es, das einmalige Anmelden von Azure AD in einer Testumgebung zu testen.

Das in diesem Tutorial beschriebene Szenario besteht aus zwei Hauptelementen:

1. Hinzufügen von Hightail aus dem Katalog
2. Konfigurieren und Testen der einmaligen Anmeldung von Azure AD


## Hinzufügen von Hightail aus dem Katalog
Zum Konfigurieren der Integration von Hightail in Azure AD müssen Sie Hightail aus dem Katalog der Liste mit den verwalteten SaaS-Apps hinzufügen.

**Um Hightail aus dem Katalog hinzuzufügen, führen Sie die folgenden Schritte aus:**

1. Klicken Sie im linken Navigationsbereich des **klassischen Azure-Portals** auf **Active Directory**. 
 
	![Active Directory][1]

2. Wählen Sie in der Liste **Verzeichnis** das Verzeichnis aus, für das Sie die Verzeichnisintegration aktivieren möchten.

3. Klicken Sie zum Öffnen der Anwendungsansicht in der oberen Menüleiste der Verzeichnisansicht auf **Anwendungen**.

	![Anwendungen][2]

4. Klicken Sie unten auf der Seite auf **Hinzufügen**.

	![Anwendungen][3]

5. Klicken Sie im Dialogfeld **Was möchten Sie tun?** auf **Anwendung aus dem Katalog hinzufügen**.

	![Anwendungen][4]

6. Geben Sie im Suchfeld als Suchbegriff **Hightail** ein.

	![Erstellen einen Azure AD-Testbenutzers](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_01.png)

7. Wählen Sie im Ergebnisbereich **Hightail** aus, und klicken Sie dann auf **Abschließen**, um die Anwendung hinzuzufügen.

	![Auswählen der App im Katalog](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_02.png)

##  Konfigurieren und Testen der einmaligen Anmeldung von Azure AD
In diesem Abschnitt soll anhand eines Testbenutzers namens Britta Simon veranschaulicht werden, wie das einmalige Anmelden von Azure AD in Hightail konfiguriert und getestet werden kann.

Damit einmaliges Anmelden funktioniert, muss Azure AD wissen, welcher Benutzer in Hightail als Gegenbenutzer zu einem Benutzer in Azure AD fungiert. Anders ausgedrückt: Zwischen einem Azure AD-Benutzer und dem entsprechenden Benutzer in Hightail muss eine Linkbeziehung eingerichtet werden.

Diese Linkbeziehung wird hergestellt, indem Sie den **Benutzernamen** in Azure AD als Wert des **Benutzernamens** in Hightail zuweisen.

Zum Konfigurieren und Testen des einmaligen Anmeldens in Azure AD mit Hightail müssen Sie die folgenden Schritte ausführen:

1. **[Konfigurieren von Azure AD – einmaliges Anmelden](#configuring-azure-ad-single-sign-on)**, um Ihren Benutzern das Verwenden dieser Funktion zu ermöglichen.
2. **[Erstellen eines Azure AD-Testbenutzers](#creating-an-azure-ad-test-user)**, um das einmalige Anmelden mit Azure AD mit dem Testbenutzer Britta Simon zu testen.
4. **[Erstellen eines Hightail-Testbenutzers](#creating-a-hightail-test-user)**, um eine Entsprechung von Britta Simon in Hightail zu erhalten, die mit ihrer Darstellung in Azure AD verknüpft ist.
5. **[Zuweisen des Azure AD-Testbenutzers](#assigning-the-azure-ad-test-user)**, um Britta Simon für das einmalige Anmelden von Azure AD zu aktivieren.
5. **[Testen der einmaligen Anmeldung](#testing-single-sign-on)**, um zu überprüfen, ob die Konfiguration funktioniert.

### Konfigurieren des einmaligen Anmeldens von Azure AD

Das Ziel dieses Abschnitts besteht darin, das einmalige Anmelden von Azure AD im klassischen Azure-Portal zu aktivieren und das einmalige Anmelden in Ihrer Hightail-Anwendung zu konfigurieren.

Die Hightail-Anwendung erwartet die SAML-Assertionen in einem bestimmten Format. Konfigurieren Sie für diese Anwendung die folgenden Ansprüche. Sie können die Werte dieser Attribute auf der Registerkarte **Attribut** der Anwendung verwalten. Der folgende Screenshot zeigt ein Beispiel für diese Attributzuordnungen:

![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_51.png)

**Führen Sie zum Konfigurieren des einmaligen Anmeldens von Azure AD bei Hightail die folgenden Schritte aus:**


1. Klicken Sie im klassischen Azure-Portal auf der Anwendungsintegrationsseite von **Hightail** im Menü oben auf **Attribute**.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_general_81.png)


1. Führen Sie im Dialogfeld **SAML-Tokenattribute** für jede Zeile in der folgenden Tabelle die folgenden Schritte aus:

	| Attributname | Attributwert |
	| --- | --- |    
	| Vorname | user.givenname |
    | Nachname | user.surname |
	| E-Mail | user.mail |
	| UserIdentity | user.mail |

	a. Klicken Sie auf **Benutzerattribut hinzufügen**, um das Dialogfeld **Benutzerattribut hinzufügen** zu öffnen.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_general_82.png)


	b. Geben Sie im Textfeld **Attributname** den für die Zeile angezeigten Attributnamen ein.

    c. Wählen Sie in der Liste **Attributwert** den für die Zeile angezeigten Attributwert aus.

    d. Klicken Sie auf **Fertig stellen**.
	



1. Klicken Sie im oberen Menü auf **Schnellstart**.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_general_83.png)



2. Wählen Sie auf der Seite **Wie sollen sich Benutzer bei Hightail anmelden?** die Option **Azure AD – einmaliges Anmelden** aus, und klicken Sie dann auf **Weiter**.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_03.png)


3. Führen Sie auf der Dialogfeldseite **App-Einstellungen konfigurieren** die folgenden Schritte aus, wenn Sie die Anwendung im **IdP-initiierten Modus** konfigurieren möchten, und klicken Sie dann auf **Weiter**:

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_04.png)



    a. Geben Sie im Textfeld **Antwort-URL** unter Verwendung des folgenden Musters die URL ein: **https://www.hightail.com/samlLogin?phi\_action=app/samlLogin&subAction=handleSamlResponse**

	b. Klicken Sie auf **Weiter**.

4. Wenn die Anwendung im **SP-initiierten Modus** konfiguriert werden soll, klicken Sie auf der Dialogfeldseite **App-Einstellungen konfigurieren** auf **Zeigen Sie die erweiterten Einstellungen an (optional)**, geben Sie die **Anmelde-URL** ein, und klicken Sie dann auf **Weiter**.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_06.png)

	a. Geben Sie im Textfeld „Anmelde-URL“ die URL ein, die von Ihren Benutzern zur Anmeldung bei der Hightail-Anwendung verwendet wird. Verwenden Sie dabei das folgende Muster: **https://www.hightail.com/loginSSO** Dies ist die allgemeine Anmeldeseite für alle Kunden, die das einmalige Anmelden verwenden möchten.

	b. Klicken Sie auf **Weiter**.

5. Führen Sie auf der Seite **Einmaliges Anmelden konfigurieren für Hightail** die folgenden Schritte aus, und klicken Sie dann auf **Weiter**:

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_05.png)


    a. Klicken Sie auf **Zertifikat herunterladen**, und speichern Sie das Base64-codierte Zertifikat auf Ihrem Computer.

    b. Klicken Sie auf **Weiter**.

	> [AZURE.NOTE] Bevor Sie das einmalige Anmelden für die Hightail-App konfigurieren, lassen Sie Ihre E-Mail-Domäne über das Hightail-Team zur Whitelist hinzufügen, damit alle Benutzer, die diese Domäne verwenden, das einmalige Anmelden nutzen können.

6. Zum Konfigurieren des einmaligen Anmeldens für Ihre Anwendung müssen Sie sich als Administrator bei Ihrem Hightail-Mandanten anmelden.

	a. Klicken Sie im Menü im oberen Bildschirmbereich auf die Registerkarte **Konto**, und wählen Sie **SAML konfigurieren**.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_001.png)


	b. Aktivieren Sie das Kontrollkästchen **SAML-Authentifizierung aktivieren**.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_002.png)

	c. Öffnen Sie das Base64-codierte Zertifikat im Editor, kopieren Sie den Inhalt des Zertifikats in die Zwischenablage, und fügen Sie ihn anschließend in das Textfeld **SAML-Tokensignaturzertifikat** ein.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_003.png)


	d. Kopieren Sie die Remoteanmelde-URL aus Azure AD in das Feld für die **SAML-Zertifizierungsstelle (Identitätsanbieter)** in Hightail.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_005.png)
	
	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_004.png)

	e. Wenn die Anwendung im **IDP-initiierten Modus** konfiguriert werden soll, wählen Sie **IdP-initiierte (Identitätsanbieter) Anmeldung**. Wenn der **SP-initiierte Modus** konfiguriert werden soll, wählen Sie **SP-initiierte (Dienstanbieter) Anmeldung**.
	
	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_006.png)

	f. Kopieren Sie die SAML-Verbraucher-URL für Ihre Instanz, und fügen Sie sie wie in Schritt 4 gezeigt in das Textfeld **Antwort-URL** ein.

	g. Klicken Sie auf **Speichern**.



6. Wählen Sie im klassischen Azure-Portal die Bestätigung zur Konfiguration der einmaligen Anmeldung aus, und klicken Sie dann auf **Weiter**.

	![Azure AD – einmaliges Anmelden][10]

7. Klicken Sie auf der Seite **Bestätigung zur einmaligen Anmeldung** auf **Fertig stellen**.
 
	![Azure AD – einmaliges Anmelden][11]




### Erstellen eines Azure AD-Testbenutzers
In diesem Abschnitt wird im klassischen Azure-Portal eine Testbenutzerin namens Britta Simon erstellt.

Wählen Sie in der Benutzerliste **Britta Simon** aus.

![Azure AD-Benutzer erstellen][20]

**Um einen Testbenutzer in Azure AD zu erstellen, führen Sie die folgenden Schritte aus:**

1. Klicken Sie im linken Navigationsbereich des **klassischen Azure-Portals** auf **Active Directory**.

	![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-hightail-tutorial/create_aaduser_09.png)

2. Wählen Sie in der Liste **Verzeichnis** das Verzeichnis aus, für das Sie die Verzeichnisintegration aktivieren möchten.

3. Klicken Sie zum Anzeigen der Liste der Benutzer im Menü oben auf **Benutzer**.
 
	![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-hightail-tutorial/create_aaduser_03.png)


4. Um das Dialogfeld **Benutzer hinzufügen** zu öffnen, klicken Sie auf der Symbolleiste unten auf **Benutzer hinzufügen**.

	![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-hightail-tutorial/create_aaduser_04.png)

5. Führen Sie auf der Dialogfeldseite **Informationen über diesen Benutzer** die folgenden Schritte aus:

	![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-hightail-tutorial/create_aaduser_05.png)

    a. Wählen Sie als **Benutzertyp** die Option **Neuer Benutzer in Ihrer Organisation** aus.

    b. Geben Sie in das Textfeld **Benutzername** den Namen **BrittaSimon** ein.

    c. Klicken Sie auf **Weiter**.

6.  Führen Sie auf der Dialogfeldseite **Benutzerprofil** die folgenden Schritte aus:

	![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-hightail-tutorial/create_aaduser_06.png)

    a. Geben Sie in das Textfeld **Vorname** den Namen **Britta** ein.

    b. Geben Sie in das Textfeld **Nachname** den Namen **Simon** ein.

    c. Geben Sie in das Textfeld **Anzeigename** den Namen **Britta Simon** ein.

    d. Wählen Sie in der Liste **Rolle** die Option **Benutzer** aus.

    e. Klicken Sie auf **Weiter**.

7. Klicken Sie auf der Dialogfeldseite **Vorübergehendes Kennwort abrufen** auf **Erstellen**.

	![Erstellen eines Azure AD-Testbenutzers](./media/active-directory-saas-hightail-tutorial/create_aaduser_07.png)


8. Führen Sie auf der Dialogfeldseite **Vorübergehendes Kennwort abrufen** die folgenden Schritte aus:
 
	![Erstellen einen Azure AD-Testbenutzers](./media/active-directory-saas-hightail-tutorial/create_aaduser_08.png)

    a. Notieren Sie den Wert von **Neues Kennwort**.

    b. Klicken Sie auf **Fertig stellen**.



### Erstellen eines Hightail-Testbenutzers

Das Ziel dieses Abschnitts ist das Erstellen eines Benutzers namens Britta Simon in Hightail.

Für Sie steht in diesem Abschnitt kein Aktionselement zur Verfügung. Hightail unterstützt die Just-In-Time-Benutzerbereitstellung auf Grundlage der benutzerdefinierten Ansprüche. Wenn Sie die benutzerdefinierten Ansprüche wie im Abschnitt **[Konfigurieren von Azure AD – einmaliges Anmelden](#configuring-azure-ad-single-sign-on)** oben konfiguriert haben, wird automatisch ein Benutzer in der Anwendung erstellt, wenn dieser noch nicht vorhanden ist.

> [AZURE.NOTE] Wenn Sie einen Benutzer manuell erstellen müssen, setzen Sie sich über [support@hightail.com](mailto:support@hightail.com) mit dem Hightail-Supportteam in Verbindung.


### Zuweisen des Azure AD-Testbenutzers

Das Ziel dieses Abschnitts besteht darin, Britta Simon die Verwendung des einmaligen Anmeldens bei Azure zu ermöglichen, indem sie Zugriff auf Hightail erhält.

![Benutzer zuweisen][200]

**Zur Zuweisung von Britta Simon zu Hightail führen Sie die folgenden Schritte aus:**

1. Klicken Sie zum Öffnen der Anwendungsansicht im klassischen Azure-Portal in der oberen Menüleiste der Verzeichnisansicht auf **Anwendungen**.
 
	![Benutzer zuweisen][201]

2. Wählen Sie in der Anwendungsliste **Hightail** aus.

	![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-hightail-tutorial/tutorial_hightail_50.png)


1. Klicken Sie im oberen Menü auf **Benutzer**.

	![Benutzer zuweisen][203]

1. Wählen Sie in der Benutzerliste **Britta Simon** aus.

2. Klicken Sie auf der Symbolleiste unten auf **Zuweisen**.

![Benutzer zuweisen][205]



### Testen der einmaligen Anmeldung

Das Ziel dieses Abschnitts ist das Testen Ihrer Azure AD-Konfiguration für einmaliges Anmelden über den Zugriffsbereich.

Wenn Sie im Zugriffsbereich auf die Kachel „Hightail“ klicken, sollten Sie automatisch bei Ihrer Hightail-Anwendung angemeldet werden.


## Zusätzliche Ressourcen

* [Liste der Tutorials zur Integration von SaaS-Apps in Azure Active Directory](active-directory-saas-tutorial-list.md)
* [Was bedeuten Anwendungszugriff und einmaliges Anmelden mit Azure Active Directory?](active-directory-appssoaccess-whatis.md)



<!--Image references-->

[1]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-hightail-tutorial/tutorial_general_205.png

<!---HONumber=AcomDC_0413_2016-->