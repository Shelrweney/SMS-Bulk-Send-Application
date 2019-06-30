package Vodafone;

import org.apache.http.HttpResponse;
import org.apache.http.*;

import com.mashape.unirest.http.Unirest;
import com.mashape.unirest.http.exceptions.UnirestException;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;

import java.io.FileOutputStream;
import java.io.FileReader;
import java.io.IOException;
import java.io.OutputStreamWriter;
import java.io.UnsupportedEncodingException;
import java.nio.file.FileAlreadyExistsException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.AddressException;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.List;
import java.util.Properties;

public class VodafoneV2 {
	public static class ConvertToSha256 {

		public static String ConvertStringToSHA256(String ReceiverMSISDN, String SMSText) {

			String secreatKey = "SecrectKey";
			String senderName = "SenderName";
			String password = "Password";
			String accountID = "AccountID";
			// String ReceiverMSISDN = "Receiver";
			// String SMSText1 = "Test SMS From SMS API by Sherief.";
			String result = "";

			String InputString = "AccountId=" + accountID + "&Password=" + password + "&SenderName=" + senderName
					+ "&ReceiverMSISDN=" + ReceiverMSISDN + "&SMSText=" + SMSText;
			// System.out.println("InputString" + InputString);

			Mac sha256_HMAC = null;
			try {

				sha256_HMAC = Mac.getInstance("HmacSHA256");

				// Encode secreat key
				byte[] bytekey = secreatKey.getBytes("UTF-8");

				// create hashed key
				SecretKeySpec secret_key = new SecretKeySpec(bytekey, "HmacSHA256");

				// Initialize the keyed hash object using the secret key as the key
				sha256_HMAC.init(secret_key);

				// Encode input string to UTF-8
				byte[] byteString = InputString.getBytes("UTF-8");

				// create Hex SHA code
				result = bytesToHexString(sha256_HMAC.doFinal(byteString)).toUpperCase();

			} catch (NoSuchAlgorithmException | InvalidKeyException | UnsupportedEncodingException e) {
				e.printStackTrace();
			}
			return result;

		}

		private static String bytesToHexString(byte[] bytes) {
			// http://stackoverflow.com/questions/332079
			StringBuffer sb = new StringBuffer();
			for (int i = 0; i < bytes.length; i++) {
				String hex = Integer.toHexString(0xFF & bytes[i]);
				if (hex.length() == 1) {
					sb.append('0');
				}
				sb.append(hex);
			}
			return sb.toString();
		}
	}

	/////////////////////////////////////////////////////////////////////////////////////////////
	public static void main(String[] args) throws UnirestException, InstantiationException, IllegalAccessException,
			ClassNotFoundException, IOException, AddressException, MessagingException {

		List<String> list1 = new ArrayList<String>();// for writing log
		String str1;
		String str2;
		String[] parts = null;
		String message;
		String En_Desc = null;
		String Ar_Desc = null;
		String locationID = null;
		String locationDSC = null;
		String Material = null;
		Statement stmt1 = null;
		Statement stmt = null;
		String DATE_FORMAT = "MM/dd/YYYY h:mm:ss";
		SimpleDateFormat sdf = new SimpleDateFormat(DATE_FORMAT);
		Calendar c1 = Calendar.getInstance();

		String DATE_FORMATLog = "yyyyMMdd";
		SimpleDateFormat sdfLog = new SimpleDateFormat(DATE_FORMATLog);
		Calendar c1Log = Calendar.getInstance();

		Connection conn = null;
		////////////////////////////////////////// Mail
		////////////////////////////////////////// setup/////////////////////////////////////////////////////////////////

		// mail setup//
		final String senderEmailID = "Any Email Address";

		// Recipient's email ID needs to be mentioned.
		String to = "Any Email Address";
		// String to1 = "Any Email Address";

		// Sender's email ID needs to be mentioned
		String from = "SMSProgram@suezcem.com";

		// Assuming you are sending email from localhost
		String host = "Email server";

		// Get system properties
		Properties properties = System.getProperties();

		// Setup mail server
		properties.setProperty("mail.smtp.host", host);
		properties.setProperty("mail.user", senderEmailID);
		properties.setProperty("mail.smtp.starttls.enable", "true");
		// properties.put("mail.smtp.auth", "true");
		// properties.setProperty("mail.user", "Any Email address");
		// properties.setProperty("mail.password", senderPassword);
		// Get the default Session object.
		// Authenticator auth = new Authenticator();
		Session session = Session.getDefaultInstance(properties);

		// Create a default MimeMessage object.

		// Session session1 = Session.getInstance(properties, auth);

		// Send message
		// Transport.send(message);
		// System.out.println("Sent message successfully....");
		// catch (MessagingException mex)
		// mex.printStackTrace();

		////////////////////////////////////////////////////////////// Database
		////////////////////////////////////////////////////////////// connection
		////////////////////////////////////////////////////////////// setup//////////////////////////////////
		try {
			Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver").newInstance();
			conn = DriverManager.getConnection(
					"jdbc:sqlserver://Ipaddress:1433;databaseName="";useUnicode=true&characterEncoding=UTF-8&useFastDateParsing=false",
					"SMS", "SMS");

			if (conn != null) {
				System.out.println("Database Successfully connected");
				// stmt = conn.createStatement();
				stmt = conn.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_READ_ONLY);
				stmt1 = conn.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_READ_ONLY);
			}

			@SuppressWarnings("unused")
			// String query = "Select * From Materials";
			// ResultSet result = stmt.executeQuery("Select * From Materials ");
			// ResultSet location = stmt1.executeQuery("Select * From Locations ");
			// } catch (SQLException e) {
			// e.printStackTrace();
			// }

			//////////////////////////////////////////////////////////////////////////////////////////////////////

			File dir = new File("ShareFolder");
			// File dir = new
			// File("//ShareFolder");
			// BufferedReader in1 = new BufferedReader(new
			// FileReader("ShareFolder/3001037018_C20181126_O20181126"));

			String[] SRC = dir.list();

			for (int i = 0; i < SRC.length; i++) {

				// Path sourcePath =
				// Paths.get("//ShareFolder/"+SRC[i]);
				Path sourcePath = Paths.get("//ShareFolder/" + SRC[i]);
				Path destinationPath = Paths
						.get("//ShareFolder/" + SRC[i]);
				list1.add(sdf.format(c1.getTime()) + ": File '" + SRC[i] + "' file copied to 'SMSArchive' folder");
				try {
					Files.copy(sourcePath, destinationPath);
				} catch (FileAlreadyExistsException e) {
					// destination file already exists
				} catch (IOException e) {
					// something else went wrong
					e.printStackTrace();
				}
			}

			for (int i = 0; i < SRC.length; i++) {
				System.out.println(SRC[i]);

				BufferedReader in1 = new BufferedReader(
						new FileReader("ShareFolder/" + SRC[i]));
				// BufferedReader in1 = new BufferedReader(new
				// FileReader("//ShareFolder/"+SRC[i]));
				while ((str1 = in1.readLine()) != null) {
					parts = str1.split(";");
					// System.out.println(s2);

					// for(int k=0;k<s2.length;k++) list1.add(s2[k]);

				}
				in1.close();

				// String[] stringArr1 = list1.toArray(new String[0]);
				for (int k = 0; k < parts.length; k++) {
					System.out.println(parts[k]);
				}
				String day = parts[5].substring(0, 4) + "." + parts[5].substring(4, 6) + "." + parts[5].substring(6, 8);
				String time = parts[6].substring(0, 2) + ":" + parts[6].substring(2, 4);
				/////////////////////////////////////////////////////////////////////////////////////////////////////
				// Connection conn=null;

				ResultSet result = stmt
						.executeQuery("Select Ar_Desc From Materials where Material = '" + parts[3] + "'");
				// ResultSet test1 = stmt.executeQuery("Select Ar_Desc From Materials where
				// Material ='"+parts[3]+"'");
				ResultSet location = stmt1
						.executeQuery("Select Ar_Desc From Locations where Location ='" + parts[4] + "'");

				// List<String> Material1 = new ArrayList<String>();
				String m_Ar_Desc;
				String l_Ar_Desc;
				int flag = 0;
				if (result.first()) {
					System.out.println("KOTB:" + result.getString("Ar_Desc"));
					m_Ar_Desc = result.getString("Ar_Desc");
				} else {
					m_Ar_Desc = parts[3];
					flag = 1;
				}

				if (location.first()) {
					System.out.println("KOTB:" + location.getString("Ar_Desc"));
					l_Ar_Desc = location.getString("Ar_Desc");
				} else {
					l_Ar_Desc = parts[4];
					flag = 2;
				}

				/*
				 * while (result.next()) { System.out.println(Ar_Desc); //Material =
				 * result.getString("Material"); //En_Desc = result.getString("En_Desc");
				 * Ar_Desc = result.getString("Ar_Desc");
				 * 
				 * 
				 * // print the results //System.out.println(Material+ "==="+En_Desc +"==="+
				 * Ar_Desc); System.out.println(Ar_Desc); //
				 * Material1.add(Material+","+En_Desc+","+Ar_Desc); }
				 * 
				 * while (location.next()) { locationID= location.getString("Location");
				 * locationDSC= location.getString("Ar_Desc"); }
				 */
				if (!parts[0].isEmpty() && parts[0].length() == 11) {
					String ReceiverMSISDN = parts[0];
					String quan = parts[2].trim();
					//message = " ØªÙ… Ø´Ø­Ù† ÙƒÙ…ÙŠØ© " + quan + " Ø·Ù† " + m_Ar_Desc + " Ù…Ù† " + l_Ar_Desc + "  " + day	+ " Ø§Ù„Ø³Ø§Ø¹Ù‡ " + time + " Ø³ÙŠØ§Ø±Ø© " + parts[7] + " Ù„Ù„Ø§Ø³ØªÙ?Ø³Ø§Ø± 19083 ";
					message = " ?? ??? ???? " +quan+" ?? "+m_Ar_Desc+ " ?? " +l_Ar_Desc+ " ??? "+ day + " ?????? " + time + " ????? " + parts[7] + " ????????? 19083 ";
					System.out.println(message);
					String SMSText = new String(message);
					new ConvertToSha256();

					////////////////////////////////////////////////////////////////////////////////////////////////////////////////
					com.mashape.unirest.http.HttpResponse<String> response = Unirest
							.post("https://e3len.vodafone.com.eg/web2sms/sms/submit/")
							.header("Content-Type", "application/xml").header("cache-control", "no-cache")

							.body("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\r\n"
									+ "<SubmitSMSRequest xmlns:=\"http://www.edafa.com/web2sms/sms/model/\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xsi:schemaLocation=\"http://www.edafa.com/web2sms/sms/model/ SMSAPI.xsd \" xsi:type=\"SubmitSMSRequest\">\r\n"
									+ "                <AccountId>""</AccountId>\r\n "
									+ "               <Password>Vodafone.1</Password>\r\n" + "<SecureHash>"
									+ ConvertToSha256.ConvertStringToSHA256(ReceiverMSISDN, SMSText)
									+ "</SecureHash>\\r\\n" + "<SMSList>\r\n"
									+ "                <SenderName>""</SenderName>\r\n" + "<ReceiverMSISDN>"
									+ parts[0] + "</ReceiverMSISDN>\r\n" + "<SMSText>" + SMSText + "</SMSText>\r\n"
									+ "</SMSList>\r\n" + "</SubmitSMSRequest>\r\n")
							.asString();
					System.out.println(response.getRawBody());
					System.out.println(SMSText);

					// File sent = new
					// File("ShareFolder/"+SRC[i]);
					// FileOutputStream fileStreamsent= new FileOutputStream(sent);
					// OutputStreamWriter writersent = new OutputStreamWriter(fileStreamsent);

					Path sourcePath = Paths.get("//ShareFolder/" + SRC[i]);
					// Path sourcePath =
					// Paths.get("//ShareFolder/"+SRC[i]);
					// Path destinationPath =
					// Paths.get("//ShareFolder/"+SRC[i]);
					Path destinationPath = Paths
							.get("ShareFolder/" + SRC[i]);
					try {
						Files.move(sourcePath, destinationPath, StandardCopyOption.REPLACE_EXISTING);
					} catch (IOException e) {
						// moving file failed.
						e.printStackTrace();
					}
					// BufferedReader in2 = new BufferedReader(new
					// FileReader("//ShareFolder/"+SRC[i]));
					// BufferedWriter smSSent =
					// new BufferedWriter (new
					// FileWriter("//ShareFolder/"+SRC[i]));

					// while((str2 = in2.readLine() ) != null){
					// smSSent.flush();
					// smSSent.write(str2);}

					// noPhone.newLine();

					// in2.close();
					// smSSent.close();
					list1.add(sdf.format(c1.getTime()) + ":File '" + SRC[i] + "' sent and  SMS has ben sent to: "
							+ parts[0] + "    " + SMSText);
					list1.add(sdf.format(c1.getTime()) + ": File '" + SRC[i] + "' sent and moved to 'SMSSent' folder ");
					MimeMessage email = new MimeMessage(session);
					email.setFrom(new InternetAddress(from));

					// Set To: header field of the header.
					email.addRecipient(Message.RecipientType.TO, new InternetAddress(to));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to1));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to2));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to3));

					// Set Subject: he ader field
					email.setSubject(" SMS has been sent to: " + parts[0]);

					// Now set the actual message
					email.setContent("File '"+ SRC[i]+ "'  moved and SMS has been sent to: "+ parts[0]+ " " +SMSText, "text/plain; charset=utf-8");
					//setText(
							//" File '" + SRC[i] + "' moved and SMS has been sent to: " + parts[0] + " " +  SMSText);

					// Send message
					Transport.send(email);
				} else {

					Path sourcePath = Paths.get("//ShareFolder/" + SRC[i]);
					// Path sourcePath =
					// Paths.get("//ShareFolder/"+SRC[i]);
					Path destinationPath = Paths
							.get("//ShareFolder/"
									+ SRC[i]);
					try {
						Files.move(sourcePath, destinationPath, StandardCopyOption.REPLACE_EXISTING);
					} catch (IOException e) {
						// moving file failed.
						e.printStackTrace();
					}
					list1.add(sdf.format(c1.getTime()) + ": File '" + SRC[i]
							+ "' without phone and moved to 'SMSwithoutPhone' folder");
				}

				if (flag == 1) {
					// send email no matrial or location found
					list1.add(sdf.format(c1.getTime()) + ": File '" + SRC[i] + "' Material not found=> " + parts[3]);
					MimeMessage email = new MimeMessage(session);
					email.setFrom(new InternetAddress(from));

					// Set To: header field of the header.
					email.addRecipient(Message.RecipientType.TO, new InternetAddress(to));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to1));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to2));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to3));

					// Set Subject: he ader field
					email.setSubject("Material not found");

					// Now set the actual message
					email.setText("Material not found=> " + parts[3]);

					// Send message
					Transport.send(email);

				} // end of if condition check location

				if (flag == 2) {
					list1.add(sdf.format(c1.getTime()) + ": File '" + SRC[i] + "' Location not found=> " + parts[4]);
					// send email no matrial or location foun
					MimeMessage email = new MimeMessage(session);
					email.setFrom(new InternetAddress(from));

					// Set To: header field of the header.
					email.addRecipient(Message.RecipientType.TO, new InternetAddress(to));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to1));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to2));
					// message1.addRecipient(Message.RecipientType.TO, new InternetAddress(to3));

					// Set Subject: he ader field
					email.setSubject("Location not found");

					// Now set the actual message
					email.setText("Location not found=> " + parts[4]);

					// Send message
					Transport.send(email);

				}

				String[] log = list1.toArray(new String[0]);

				File fo = new File("//ShareFolder/Log"
						+ sdfLog.format(c1Log.getTime()) + ".txt");
				FileOutputStream fileStream = new FileOutputStream(fo, true);
				OutputStreamWriter writer = new OutputStreamWriter(fileStream, "UTF8");

				BufferedWriter logfilein = new BufferedWriter(writer);

				for (String line : log) {
					logfilein.write(line);
					logfilein.newLine();
				}
				logfilein.close();

			} // general for

			// BufferedReader in3 = new BufferedReader(new
			// FileReader("//ShareFolder/"+SRC[i]));

			// BufferedWriter noPhone =
			// new BufferedWriter (new
			// FileWriter("//ShareFolder/SMSwithoutPhone/"+SRC[i]));

			// while((str2 = in3.readLine() ) != null){
			// noPhone.write(str2);}
			// for (String line : parts ){
			// noPhone.write(line);
			// noPhone.newLine();
			// }
			// in3.close();
			// noPhone.close ();

			// } //end of else for empty mobile
			// End of while loop for Material

			// String[] Material2 = Material1.toArray(new String[0]);

			// Delete

			// File file = new
			// File("//ShareFolder/SMS/"+SRC[i]);
			// RandomAccessFile raf=new RandomAccessFile(file,"rw");
			// raf.close();
			// if(file.delete())
			// {
			// System.out.println("File deleted successfully");
			// list1.add(c1.getTime() +": File "+SRC[i]+" deleted successfully");
			//
			// }
			// else
			// {
			// System.out.println("Failed to delete the file");
			// list1.add(c1.getTime() +": File "+SRC[i]+" Failed to delete");
			// }
			// for (int i=0;i<Material2.length;i++) {System.out.println(Material2[i]);}
			// System.out.println(result);
			// if (!Material.equals(parts[3])) { System.out.println("This is part3 "+
			// parts[3]);}
			/////////////////////////////////////////////////////////////////////////////////////////////////////////////////

		} // end of general For
		catch (SQLException e) {
			e.printStackTrace();
		}
	}
	//////////////////////////////////////////////////////////////////////////////////////////////////////

}
