package com.example.gptchatmicroservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.io.FileWriter;
import java.io.IOException;

@SpringBootApplication
@RestController
public class ChatGPTervice {
    private static final String API_URL = "https://chat.openai.com/chat";
    private static final String API_KEY = "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";

    public static void main(String[] args) {
        SpringApplication.run(ChatGPTervice.class, args);
    }

    @PostMapping(value = "/chatgpt")
    public String chatWithGPT(@RequestBody String question) {
        RestTemplate restTemplate = new RestTemplate();

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.add("Authorization", "Bearer " + API_KEY);

        MultiValueMap<String, String> requestMap = new LinkedMultiValueMap<>();
        requestMap.add("model", "text-davinci-003");
        requestMap.add("prompt", question);
        requestMap.add("max_tokens", "4000");
        requestMap.add("temperature", "1.0");

        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(requestMap, headers);
        String response = restTemplate.postForObject(API_URL, request, String.class);

        ObjectMapper mapper = new ObjectMapper();
        JsonNode root = null;
        try {
            root = mapper.readTree(response);
        } catch (IOException e) {
            e.printStackTrace();
        }

        JsonNode answerNode = root.path("choices").get(0).path("text");
        String answer = answerNode.asText();

        try (FileWriter fileWriter = new FileWriter("database.csv", true)) {
            fileWriter.append(question).append(";").append(answer).append("\n");
        } catch (IOException e) {
            e.printStackTrace();
        }

        return answer;
    }
}
