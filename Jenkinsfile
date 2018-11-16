#!groovy
import net.sf.json.JSONArray;
import net.sf.json.JSONObject
node {
    JSONArray attachments = new JSONArray();
    JSONObject attachment = new JSONObject();

    attachment.put('text','I find your lack of faith disturbing!');
    attachment.put('fallback','Hey, Vader seems to be mad at you.');
    attachment.put('color','#ff0000');

    attachments.add(attachment); 
    slackSend(color: '#00FF00', channel: '@gustavo.maia', attachments: attachments.toString())
}
