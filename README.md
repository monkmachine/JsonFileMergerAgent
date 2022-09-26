package org.dacogb.jsonFileMerger;

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.stream.IntStream;

import com.fasterxml.jackson.core.JsonEncoding;
import com.fasterxml.jackson.core.JsonFactory;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonGenerator.Feature;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.TreeNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class JsonFileMerger {

	private JsonFactory jfactory = new JsonFactory();
	private String OutputFolder;
	private String ArrayField;
	private String InputFolder;
	private JsonEncoding JSONEncoding = JsonEncoding.UTF8;
	private ObjectMapper Mapper = new ObjectMapper();
	private JsonGenerator jGenerator;
	

	public void execute() throws IOException, IllegalArgumentException {
		//FileOutputStream fileOut =new FileOutputStream(OutputFolder + "/" + ArrayField + ".json");
		BufferedOutputStream fileOut = new BufferedOutputStream(new FileOutputStream(OutputFolder + "/" + ArrayField + ".json"), 32768);
		jGenerator = jfactory.createGenerator(fileOut, JSONEncoding);
		jGenerator.enable(Feature.FLUSH_PASSED_TO_STREAM);
		jGenerator.writeStartObject();
		jGenerator.writeFieldName(ArrayField);
		jGenerator.writeStartArray();
		IntStream s = IntStream.range(0, 20);
		try {
			Files.walk(Paths.get(InputFolder)).parallel().filter(p -> p.toString().endsWith(".json")).forEach(p -> {
				try {

					InputStream jsonStream = new FileInputStream(p.toFile());
					JsonParser jsonParser = new JsonFactory().createParser(jsonStream);
					TreeNode treeNode = Mapper.readTree(jsonParser);
					writejson(treeNode);
					jsonStream.close();
					jsonParser.close();

				} catch (IOException e) {
					e.printStackTrace();
				}
			});

		} catch (Exception e) {
			e.printStackTrace();
		}
		jGenerator.writeEndArray();
		jGenerator.close();
		fileOut.close();

	}
	private synchronized void writejson(TreeNode treeNode) throws IOException {
		jGenerator.setCodec(Mapper);
		jGenerator.writeTree(treeNode);
	}
	
	public void setOutputFolder(String outputFolder) {
		OutputFolder = outputFolder;
	}

	public void setArrayField(String arrayField) {
		ArrayField = arrayField;
	}

	public void setInputFolder(String inputFolder) {
		InputFolder = inputFolder;
	}

	public void setJSONEncoding(String jSONEncoding) {

		if (jSONEncoding == "UTF8") {
			JSONEncoding = JsonEncoding.UTF8;
		} else if (jSONEncoding == "UTF16_BE") {
			JSONEncoding = JsonEncoding.UTF16_BE;
		} else if (jSONEncoding == "UTF16_LE") {
			JSONEncoding = JsonEncoding.UTF16_LE;
		} else if (jSONEncoding == "UTF32_BE") {
			JSONEncoding = JsonEncoding.UTF32_BE;
		} else if (jSONEncoding == "UTF32_LE") {
			JSONEncoding = JsonEncoding.UTF32_LE;
		}
	}

	private void validateConstructor() {
		if (InputFolder == null) {
			throw new IllegalArgumentException("InputFolder");
		}
		if (ArrayField == null) {
			throw new IllegalArgumentException("ArrayField");
		}
		if (OutputFolder == null) {
			throw new IllegalArgumentException("OutputFolder");
		}
	}

}
