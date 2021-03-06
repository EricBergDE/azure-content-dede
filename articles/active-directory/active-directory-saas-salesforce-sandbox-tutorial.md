<properties 
    pageTitle="Tutorial: Azure Active Directory-Integration mit Salesforce Sandbox | Microsoft Azure"
    description="Erfahren Sie, wie Sie Salesforce Sandbox mit Azure Active Directory verwenden können, um einmalige Anmeldung, automatisierte Bereitstellung und mehr zu aktivieren!" 
    services="active-directory" 
    authors="jeevansd"  
    documentationCenter="na" 
    manager="stevenpo"/>

<tags 
    ms.service="active-directory" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="identity" 
    ms.date="05/19/2016" 
    ms.author="jeedes" />


#Tutorial: Azure Active Directory-Integration mit Salesforce Sandbox
>[AZURE.TIP]Klicken Sie [hier](http://go.microsoft.com/fwlink/?LinkId=521878), um Feedback abzugeben.
  
In diesem Tutorial wird die Integration von Azure und Salesforce Sandbox erläutert.  
Sandboxes bieten Ihnen die Möglichkeit, mehrere Kopien Ihres Unternehmens in separaten Umgebungen für eine Vielzahl von Zwecken, wie z. B. Entwicklung, Tests und Schulungen, ohne eine Beeinträchtigung der Daten und Anwendungen in Ihrer Vertriebsproduktions-Organisation zu erstellen. 
Weitere Informationen finden Sie unter [Sandbox-Übersicht](https://help.salesforce.com/HTViewHelpDoc?id=create_test_instance.htm&language=en_US)
  
Das in diesem Lernprogramm verwendete Szenario setzt voraus, dass Sie bereits über die folgenden Elemente verfügen:

-   Ein gültiges Azure-Abonnement
-   Eine Sandbox unter Salesforce.com
  
Wenn Sie noch nicht über eine gültige Sandbox unter Salesforce.com verfügen, müssen Sie sich an Salesforce wenden.
  
Das in diesem Lernprogramm beschriebene Szenario besteht aus den folgenden Bausteinen:

1.  Aktivieren der Anwendungsintegration für Salesforce Sandbox
2.  Konfigurieren der einmaligen Anmeldung
3.  Aktivieren Ihrer Domäne
4.  Konfigurieren der Benutzerbereitstellung
5.  Zuweisen von Benutzern

![Szenario](./media/active-directory-saas-salesforce-sandbox-tutorial/IC769571.png "Szenario")
##Aktivieren der Anwendungsintegration für Salesforce Sandbox
  
In diesem Abschnitt wird beschrieben, wie Sie die Anwendungsintegration für Salesforce Sandbox aktivieren.

###So aktivieren Sie die Anwendungsintegration für Salesforce Sandbox:

1.  Klicken Sie im klassischen Azure-Portal im linken Navigationsbereich auf **Active Directory**.

    ![Active Directory](./media/active-directory-saas-salesforce-sandbox-tutorial/IC700993.png "Active Directory")

2.  Wählen Sie in der Liste **Verzeichnis** das Verzeichnis aus, für das Sie die Verzeichnisintegration aktivieren möchten.

3.  Klicken Sie zum Öffnen der Anwendungsansicht in der oberen Menüleiste der Verzeichnisansicht auf **Anwendungen**.

    ![Anwendungen](./media/active-directory-saas-salesforce-sandbox-tutorial/IC700994.png "Anwendungen")

4.  Klicken Sie zum Öffnen des **Anwendungskatalogs** auf **App hinzufügen** und anschließend auf **Eine Anwendung hinzufügen, die mein Unternehmen verwendet**.

    ![Was möchten Sie tun?](./media/active-directory-saas-salesforce-sandbox-tutorial/IC700995.png "Was möchten Sie tun?")

5.  Geben Sie in das **Suchfeld** **Salesforce Sandbox** ein.

    ![Anwendungskatalog](./media/active-directory-saas-salesforce-sandbox-tutorial/IC710978.png "Anwendungskatalog")

6.  Wählen Sie im Ergebnisbereich **Salesforce Sandbox** aus, und klicken Sie dann auf **Abschließen**, um die Anwendung hinzuzufügen.

    ![Salesforce Sandbox](./media/active-directory-saas-salesforce-sandbox-tutorial/IC746474.png "Salesforce Sandbox")
##Konfigurieren der einmaligen Anmeldung
  
In diesem Abschnitt wird erläutert, wie Sie es Benutzern mithilfe einer Verbundanmeldung auf Basis des SAML-Protokolls ermöglichen, sich mit ihrem Azure AD-Konto bei Salesforce Sandbox zu authentifizieren.

###So konfigurieren Sie einmaliges Anmelden

1.  Klicken Sie im klassischen Azure-Portal auf der Anwendungsintegrationsseite für **Salesforce Sandbox** auf **Einmaliges Anmelden konfigurieren**, um das Dialogfeld **Einmaliges Anmelden konfigurieren** zu öffnen.

    ![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-salesforce-sandbox-tutorial/IC749323.png "Einmaliges Anmelden konfigurieren")

2.  Wählen Sie auf der Seite **How would you like users to sign on to Salesforce Sandbox** die Option **Microsoft Azure AD – einmaliges Anmelden** aus, und klicken Sie dann auf **Weiter**.

    ![Salesforce Sandbox](./media/active-directory-saas-salesforce-sandbox-tutorial/IC746479.png "Salesforce Sandbox")

3.  Geben Sie auf der Seite **App-URL konfigurieren** im Textfeld **Anmelde-URL** die URL im Format `http://company.my.salesforce.com` ein, und klicken Sie dann auf **Weiter**.

    ![App-URL konfigurieren](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781022.png "App-URL konfigurieren")

4. Wenn Sie bereits einmaliges Anmelden für eine andere Instanz von Salesforce Sandbox in Ihrem Verzeichnis konfiguriert haben, müssen Sie den **Bezeichner** auf den gleichen Wert wie die **Anmelde-URL** konfigurieren. Das Feld **Bezeichner** wird angezeigt, wenn Sie das Kontrollkästchen **Erweiterte Einstellungen anzeigen** auf der Seite **App-URL konfigurieren** des Dialogfelds aktivieren.

4.  Klicken Sie auf der Seite **Einmaliges Anmelden konfigurieren um Salesforce Sandbox** auf **Zertifikat herunterladen**, und speichern Sie das Zertifikat auf Ihrem Computer.

    ![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781023.png "Einmaliges Anmelden konfigurieren")

5.  Melden Sie sich in einem anderen Webbrowserfenster bei der Salesforce Sandbox-Unternehmenswebsite als Administrator an.

6.  Klicken Sie im oberen Menü auf **Setup**.

    ![Einrichtung](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781024.png "Einrichtung")

7.  Klicken Sie auf der Salesforce Sandbox-Unternehmenswebsite auf **Security Controls** und anschließend auf **Einzelanmeldungseinstellungen**.

    ![Einstellungen für einmaliges Anmelden](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781025.png "Einstellungen für einmaliges Anmelden")

8.  Führen Sie im Abschnitt „Einstellungen für einmaliges Anmelden“ die folgenden Schritte aus:

    ![Einstellungen für einmaliges Anmelden](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781026.png "Einstellungen für einmaliges Anmelden")

    a. Wählen Sie **SAML aktiviert** aus.
    
    b. Klicken Sie auf **Neu**.

9.  Führen Sie im Abschnitt „Einstellungen für einmalige Anmelden für SAML“ die folgenden Schritte aus:

    ![SAML-Einstellungen für einmaliges Anmelden](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781027.png "SAML-Einstellungen für einmaliges Anmelden")

    a. Geben Sie im Textfeld "Name" den Namen der Konfiguration (z. B.: *SPSSOWAAD\_Test*) ein.
    
    b. Kopieren Sie im klassischen Azure-Portal auf der Dialogseite **Einmaliges Anmelden konfigurieren um Salesforce Sandbox** den Wert für **Aussteller-URL**, und fügen Sie ihn in das Textfeld **Aussteller** ein.
    
    c. Geben Sie im Textfeld **Entitäts-ID** die Zeichenfolge **https://test.salesforce.com** ein, wenn dies die erste Salesforce Sandbox-Instanz ist, die Sie Ihrem Verzeichnis hinzufügen. Wenn Sie bereits eine Instanz der Salesforce Sandbox hinzugefügt haben, geben Sie die **Entitäts-ID** die **Anmelde-URL** im folgenden Format ein: `http://company.my.salesforce.com`
    
    d. Klicken Sie auf **Durchsuchen**, um das heruntergeladene Zertifikat hochzuladen.
    
    e. Wählen Sie für **SAML Identity Type** die Option **Assertion contains the Federation ID from the User object** aus.
    
    f. Wählen Sie als **SAML Identity Location** die Option **Identity is in the NameIdentifier element of the Subject statement** aus.
    
    g. Kopieren Sie im klassischen Azure-Portal auf der Dialogseite **Einmaliges Anmelden konfigurieren um Salesforce Sandbox** den Wert für **Remoteanmelde-URL**, und fügen Sie ihn in das Textfeld **Anmelde-URL des Identitätsanbieters** ein.
    
    h. Kopieren Sie im klassischen Azure-Portal auf der Dialogseite **Einmaliges Anmelden konfigurieren um Salesforce Sandbox** den Wert für **Remoteabmelde-URL**, und fügen Sie ihn in das Textfeld **Abmelde-URL des Identitätsanbieters** ein.
    
    i. Wählen Sie für **Service Provider Initiated Request Binding** die Option **HTTP Post** aus.
    
    j. Klicken Sie auf **Speichern**.

10. Wählen Sie im klassischen Azure-Portal die Bestätigung zur Konfiguration des einmaligen Anmeldens aus, und klicken Sie dann auf **Abschließen**, um das Dialogfeld **Einmaliges Anmelden konfigurieren** zu schließen.

    ![Einmaliges Anmelden konfigurieren](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781028.png "Einmaliges Anmelden konfigurieren")

##Aktivieren Ihrer Domäne
  
In diesem Abschnitt wird davon ausgegangen, dass Sie bereits eine Domäne erstellt haben.  
Weitere Informationen finden Sie unter [Define Your Domain Name](https://help.salesforce.com/HTViewHelpDoc?id=domain_name_define.htm&language=en_US) (in englischer Sprache).

###Führen Sie zum Aktivieren Ihrer Domäne die folgenden Schritte aus:

1.  Klicken Sie im linken Navigationsbereich auf **Domänenverwaltung**, und klicken Sie dann auf **My Domain**.

    ![Meine Domäne](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781029.png "Meine Domäne")

    >[AZURE.NOTE]Stellen Sie sicher, dass Ihre Domäne richtig konfiguriert wurde.

2.  Klicken Sie im Abschnitt **Login Page Settings** auf **Bearbeiten**, und wählen Sie dann als **Authentifizierungsdienst** den Namen der SAML-Einstellung zum einfachen Anmelden aus dem vorherigen Abschnitt aus. Klicken Sie abschließend auf **Speichern**.

    ![Meine Domäne](./media/active-directory-saas-salesforce-sandbox-tutorial/IC781030.png "Meine Domäne")
  
Sobald Sie eine Domäne konfiguriert haben, sollten Ihre Benutzer die Domänen-URL für die Anmeldung der Salesforce Sandbox verwenden. Wenn den Wert der URL abrufen möchten, klicken Sie auf das SSO-Profil, das Sie im vorherigen Abschnitt erstellt haben.
##Konfigurieren der Benutzerbereitstellung
  
In diesem Abschnitt wird erläutert, wie Sie die Bereitstellung von Active Directory-Benutzerkonten für Salesforce Sandbox aktivieren.

###So konfigurieren Sie die Benutzerbereitstellung

1.  Wählen Sie im Salesforce-Portal in der oberen Navigationsleiste Ihren Namen aus, um Ihr Benutzermenü zu erweitern:

    ![Meine Einstellungen](./media/active-directory-saas-salesforce-sandbox-tutorial/IC698773.png "Meine Einstellungen")

2.  Wählen Sie in Ihrem Benutzermenü die Option **Meine Einstellungen** aus, um die Seite **Meine Einstellungen** zu öffnen.

3.  Klicken Sie im linken Navigationsbereich auf **Persönlich**, um den entsprechenden Abschnitt zu erweitern, und dann auf **Reset My Security Token**:

    ![Meine Einstellungen](./media/active-directory-saas-salesforce-sandbox-tutorial/IC698774.png "Meine Einstellungen")

4.  Klicken Sie auf der Seite **Reset My Security Token** auf **Reset Security Token**, um eine E-Mail mit Ihrem Salesforce.com-Sicherheitstoken anzufordern.

    ![Neues Token](./media/active-directory-saas-salesforce-sandbox-tutorial/IC698776.png "Neues Token")

5.  Prüfen Sie Ihren Posteingang auf eine E-Mail von Salesforce.com mit "**salesforce.com.com Security Bestätigung**" als Betreff.

6.  Öffnen Sie die E-Mail, und kopieren Sie das Sicherheitstoken.

7.  Klicken Sie im klassischen Azure-Portal auf der Anwendungsintegrationsseite für **Salesforce Sandbox** auf **Benutzerbereitstellung konfigurieren**, um das Dialogfeld **Benutzerbereitstellung konfigurieren** zu öffnen.

    ![Konfigurieren der Benutzerbereitstellung](./media/active-directory-saas-salesforce-sandbox-tutorial/IC769573.png "Konfigurieren der Benutzerbereitstellung")

8.  Legen Sie auf der Seite **Geben Sie Ihre Salesforce Sandbox Anmeldeinformationen ein, um die automatische Benutzerbereitstellung zu aktivieren** die folgenden Konfigurationseinstellungen fest:

    ![Salesforce Sandbox](./media/active-directory-saas-salesforce-sandbox-tutorial/IC746476.png "Salesforce Sandbox")

    a. Geben Sie im Textfeld **Administratorbenutzername für Salesforce Sandbox** den Namen eines Salesforce Sandbox-Kontos ein, dem das Profil **Systemadministrator** in "Salesforce.com" zugewiesen ist.

    b. Geben Sie im Feld **Salesforce Sandbox-Administratorkennwort** das Kennwort für das Konto ein.

    c. Fügen Sie das Sicherheitstoken in das Textfeld **Benutzersicherheitstoken** ein.

    d. Klicken Sie auf **Überprüfen**, um die Konfiguration zu überprüfen.

    e. Klicken Sie auf **Weiter**, um die Bestätigungsseite zu öffnen.

9.  Klicken Sie auf der Bestätigungsseite auf **Abgeschlossen**, um die Konfiguration zu speichern.
##Zuweisen von Benutzern
  
Um Ihre Konfiguration zu testen, müssen Sie den Azure AD-Benutzern, denen Sie die Verwendung Ihrer Anwendung ermöglichen möchten, Zugriff auf die Anwendung gewähren. Weisen Sie dazu der Anwendung Benutzer zu.

###So weisen Sie Salesforce Sandbox Benutzer zu:

1.  Erstellen Sie im klassischen Azure-Portal ein Testkonto.

2.  Klicken Sie auf der Anwendungsintegrationsseite für Salesforce Sandbox auf **Benutzer zuweisen**.

    ![Benutzer zuweisen](./media/active-directory-saas-salesforce-sandbox-tutorial/IC769574.png "Benutzer zuweisen")

3.  Wählen Sie den Testbenutzer aus, klicken Sie auf **Zuweisen** und anschließend auf **Ja**, um die Zuweisung zu bestätigen.

    ![Ja](./media/active-directory-saas-salesforce-sandbox-tutorial/IC767830.png "Ja")
  
Nach 10 Minuten können Sie überprüfen, ob das Konto mit Salesforce Sandbox synchronisiert wurde.
  
Wenn Sie die Einstellungen für einmaliges Anmelden testen möchten, öffnen Sie den Zugriffsbereich. Weitere Informationen zum Zugriffsbereich finden Sie unter [Einführung in den Zugriffsbereich](https://msdn.microsoft.com/library/dn308586).

<!---HONumber=AcomDC_0525_2016-->