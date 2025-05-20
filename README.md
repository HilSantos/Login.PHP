# Login.PHP
Criação do login.php

<?php
include('segurancazero.php');
include('conn.php');
if($_SERVER['REQUEST_METHOD']=='POST'){
    $email = $_POST['email'];
    $senha = $_POST['senha'];
    $sql = "SELECT COUNT(*) FROM tb_usuarios WHERE email_usuario = '$email'
    AND senha_usuario = '$senha'";
    $result = mysqli_query($link,$sql);
    $count = mysqli_fetch_array($result);
    if($count[0] == 1){
        //email e senha corretos//
        $sql = "SELECT id_usuario, apelido_usuario, nivel_usuario
        FROM tb_usuarios WHERE email_usuario = '$email' AND senha_usuario = '$senha'";
        echo$sql;
        $result = mysqli_query($link,$sql);
        $tbl = mysqli_fetch_array($result);
        $_SESSION['id_usuario'] = $tbl[0];
        $_SESSION['apelido'] = $tbl[1];
        $_SESSION['nivel'] = $tbl[2];
        mysqli_close($link);
        header('Location: index.php');
        exit();
    }
    else{
        mysqli_close($link);
        header('Location: login.php?msg=Usuario e/ou senha invalidos!');
        exit();
    }
}

?>

<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="lista.css">
    <title>Login</title>
</head>
<body>
    <br>
    <h1>Login</h1>
    <?php
    include('msg_user.php');
    ?>
    <br><br>
    <form action="login.php" method="post">
        <label for="email">Email</label>
        <input type="email" name="email" id="email" required>
        <br>
        <label for="senha">Senha</label>
        <input type="password" name="senha" id="senha" required>
        <br><br>
        <input type="submit" value="Entrar">
    </form>
</body>
</html>
